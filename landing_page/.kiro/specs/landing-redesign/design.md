# CivicSense Landing Page Redesign — Design

## 0. Context & constraints

- **Stage B decisions (override Stage A where they conflict)**: (1) KEEP Election Calendar + Politician Watch as roadmap adverts labeled "Coming soon" — do not build them, never claim they are live. (2) Metrics are aspirational GOALS, reworded as aims, never achieved results. (3) KEEP the owner's copy generally; unshipped things are "coming soon"/goals, not deleted. (4) Team section KEPT but with RANDOM/PLACEHOLDER names (no real handles/personal names) for anonymity. (5) The owner-supplied two-paragraph "What is CivicSense" copy is authoritative for the About section. The hero pipeline visual is unchanged and still depicts only the real WhatsApp fact-check loop.
- **Surface**: `/projects/sandbox/civic_sense/landing_page` only. Do not touch sibling apps or repo-root files.
- **Stack (fixed, from `package.json`)**: Next.js `16.2.6` (App Router, `typedRoutes: true`, `output: "standalone"`), React `19.2.6`, TypeScript `5.9.3`, Tailwind CSS `4.3.3` via `@tailwindcss/postcss` (CSS-first config in `src/app/globals.css`; **no** `tailwind.config` file), `motion ^12.43.0`, Radix UI (`@radix-ui/react-dialog`, `radix-ui`, `@radix-ui/react-icons`), `vaul` (drawer), `lucide-react`, `class-variance-authority` + `clsx` + `tailwind-merge` (`cn` in `src/lib/utils.ts`), `tw-animate-css`. Lint/format via Biome `2.5.9` (`biome.jsonc`). shadcn "new-york" style (`components.json`).
- **Note**: `components.json` points `tailwind.css` at `src/globals.css`, but the real file is `src/app/globals.css`. This is a cosmetic mismatch (shadcn CLI hint only); the app imports `@/app/globals.css` in `layout.tsx`. Do not "fix" it as part of the redesign unless adding shadcn components; if touched, correct it to `src/app/globals.css`.
- **Network**: sandbox is repository-access-only. The external reference repos in the brief (react-three-fiber, ybouane/liquidglass, naughtyduk/liquidGL, dashersw/liquid-glass-js, rizzytoday/liquid-glass) could **not** be fetched in this environment (HTTP 403). This design therefore relies on well-documented, stable techniques of those libraries and standard CSS/WebGL rather than their source. Implementation stages should re-attempt fetch if network allows; none of the decisions below require fetching them.

## 1. Design direction (visual language)

Target adjectives: premium, civic, trustworthy, intelligent, calm, editorial, spatial, intentional. Anti-goals: generic AI-SaaS gradients, purple/blue AI cliché, glass-everywhere, card-stack monotony, noisy particles, gratuitous 3D.

Direction: **"Verified civic instrument."** Dark, editorial base with a single confident emerald accent (already the product's color) and a deep navy atmosphere pulled from the existing hero overlay. Depth via layered surfaces and restrained glass, not blobs. Motion is choreographic and causal (evidence converging into a verdict), never ambient noise.

### 1.1 Color tokens
Derived only from colors already in the repo (emerald `#2ECC71`, `#34d399`, `rgb(16 185 129)`; navy `rgba(13,27,42,*)`; existing oklch neutrals). Add a semantic layer in `:root` while preserving all existing shadcn tokens.

```
/* Brand + semantic (new) */
--background:        oklch(0.145 0 0);            /* keep existing near-black */
--foreground:        oklch(0.985 0 0);
--surface:           oklch(0.205 0 0);            /* = existing --card */
--surface-elevated:  oklch(0.245 0 0);
--muted:             oklch(0.269 0 0);            /* keep */
--muted-foreground:  oklch(0.708 0 0);            /* keep */
--border:            oklch(1 0 0 / 10%);          /* keep */
--accent:            #2ECC71;                     /* emerald brand */
--accent-strong:     #34d399;
--accent-soft:       rgb(16 185 129 / 0.15);
--accent-secondary:  #0d1b2a;                     /* civic navy */
--success:           #34d399;
--warning:           #f59e0b;                     /* amber (verdict MISLEADING) */
--danger:            oklch(0.704 0.191 22.216);   /* = existing --destructive */
```
Verdict palette (already used, formalize as tokens): VERIFIED emerald, MISLEADING amber, FALSE red, UNVERIFIED neutral. Keep `--primary`, `--card`, `--popover`, `--ring`, `--radius`, chart/sidebar tokens untouched so `ui/button.tsx` and `ui/popover.tsx` are unaffected.

Gradients: reserved for hero atmosphere, glass highlights, interactive states, and one section transition. No full-page gradient.

### 1.2 Type scale (Inter, already loaded via `next/font/google`)
Keep Inter; introduce a disciplined scale via tokens/utility classes. Tight line-height on display, comfortable measure (~60–70ch) on body.

```
display-xl  clamp(2.75rem, 6vw, 6rem)   / lh 0.95 / tracking -0.03em  (hero H1)
display-lg  clamp(2rem, 4vw, 3.25rem)   / lh 1.0  / tracking -0.02em  (section H2)
title       1.25–1.5rem / lh 1.15
body-lg     1.125–1.25rem / lh 1.6 / measure ~65ch
body        1rem / lh 1.6
eyebrow     0.75rem / uppercase / tracking 0.18em / accent color
mono-num    tabular-nums for any real numeric stat
```
Do not scale text up merely for fashion; hero hierarchy must read in ~2–3s.

### 1.3 Spacing, radius, elevation
- Spacing scale (map to Tailwind + a small constants set): 4, 8, 12, 16, 24, 32, 48, 64, 96, 128 px. Section vertical rhythm ~`py-24`/`py-32` on desktop, tighter on mobile; keep `container max-w-7xl px-8` (already defined).
- Radius: reuse `--radius: 0.65rem` base and existing sm/md/lg/xl derivations; add `--radius-2xl` for large glass panels.
- Elevation: 3 levels (resting, hover, floating) as shadow tokens; glass adds inset top highlight + soft drop.

### 1.4 Motion language
- Timings: `--motion-fast 150ms`, `--motion-base 250ms`, `--motion-slow 450ms`, `--motion-cinematic 900ms`. Easing: standard `cubic-bezier(0.4, 0, 0.2, 1)`; entrances use a soft ease-out; springs only for tactile press (subtle, low bounce).
- Hero: staged entrance (eyebrow -> headline -> paragraph -> CTAs -> trust row), ≤ ~900ms total.
- Nav: soft transparent->glass on scroll.
- Cards: small hover elevation + border warm-up (emerald), no large translate.
- Marquees: constant linear; **pause on hover/focus and offscreen**.
- Product viz: causal reveal (claim -> three sources converge -> single verdict), loop only while visible.
- All gated by `prefers-reduced-motion` and IntersectionObserver.

## 2. Architecture & component structure

Keep `page.tsx` as thin composition. Reorganize components into intent folders; keep names stable where possible to minimize churn. Proposed target layout under `src/components/`:

```
components/
  layout/       site-header.tsx, site-footer.tsx, section.tsx (shared wrapper: id, eyebrow, heading, spacing)
  hero/         hero-section.tsx, hero-media.tsx (next/image slideshow), pipeline-visual.tsx
  sections/     about-section.tsx, sources-ticker.tsx, how-it-works.tsx, sample-verdicts.tsx, team-section.tsx, call-to-action.tsx
  visual/       pipeline-visual.tsx (DOM/SVG default); OPTIONAL: civic-scene.tsx, scene-fallback.tsx (R3F, only if gated + justified)
  glass/        glass-panel.tsx (the reusable primitive), glass.css or tokens in globals.css
  motion/       use-in-view-pause.ts, use-reduced-motion helpers, motion-tokens.ts
  ui/           button.tsx, popover.tsx (existing, keep)
  action-button.tsx, icons.tsx (keep)
```

Folder moves are optional if they add risk; the mandatory deliverables are the **glass primitive**, the **pipeline visual**, the **token layer**, the **offscreen/reduced-motion motion helpers**, and the **copy corrections**. Do not create dozens of trivial components. If moving files, update the `@/` imports (tsconfig path alias `@/* -> ./src/*`).

### 2.1 Shared primitives to add
- **`GlassPanel`** (client or server-safe wrapper): renders a surface with the glass tiers (§3). Props: `as`, `elevation`, `interactive`, `className`. Detects support/tier at runtime for the interactive tier; SSR renders the safe fallback to avoid hydration mismatch, then upgrades in `useEffect`.
- **`Section`**: consistent `id`, eyebrow, `h2`, description, and vertical rhythm; centralizes spacing so sections stop redefining `py-20 md:py-24`.
- **`useInViewPause`**: wraps IntersectionObserver to expose `isInView`; callers pause intervals/rAF/marquee when false.
- **`motion-tokens.ts`**: exports durations/easings/variants so timings aren't duplicated.

## 3. Glass / material strategy (progressive enhancement)

Single primitive, three tiers, chosen at runtime with SSR-safe defaults.

- **Tier A — Enhanced (capable desktop, motion allowed)**: `backdrop-filter: blur(20px) saturate(130%)`, layered gradient fill, 1px light border, inset top highlight, soft drop shadow, and a subtle edge-light that responds to pointer within the panel (optional, cheap, `motion` value; no per-frame layout). Optional SVG displacement filter (`feTurbulence`+`feDisplacementMap`) for a refraction hint on hero chrome only — kept very low strength and behind a support check.
- **Tier B — Reduced motion**: identical static glass, no pointer response, no displacement animation.
- **Tier C — Fallback (no `backdrop-filter` support, or low-power/mobile heuristic)**: solid `--surface`/`--surface-elevated` with border + subtle shadow; no blur. Detect via `CSS.supports('backdrop-filter','blur(1px)')` (and `-webkit-`).

Baseline CSS (adapted to CivicSense tokens, not copied wholesale from any repo):

```css
.glass {
  position: relative; isolation: isolate; overflow: hidden;
  background: linear-gradient(135deg, rgb(255 255 255 / 0.10), rgb(255 255 255 / 0.04));
  border: 1px solid rgb(255 255 255 / 0.14);
  box-shadow: inset 0 1px 0 rgb(255 255 255 / 0.16), 0 20px 60px rgb(0 0 0 / 0.35);
  backdrop-filter: blur(20px) saturate(130%);
  -webkit-backdrop-filter: blur(20px) saturate(130%);
}
.glass::before { /* top-left sheen */ content:""; position:absolute; inset:0; pointer-events:none;
  background: linear-gradient(135deg, rgb(255 255 255 / 0.14), transparent 35%, transparent 70%, rgb(255 255 255 / 0.05)); }
@supports not ((backdrop-filter: blur(1px)) or (-webkit-backdrop-filter: blur(1px))) {
  .glass { background: var(--surface-elevated); }
}
```
Contrast: text over glass must keep AA; where imagery sits behind (hero), keep the existing navy scrim (`rgba(13,27,42,*)`) or increase fallback opacity. Glass is decorative only — legibility never depends on it.

## 4. Product visualization (the real pipeline)

**Decision: DOM + SVG + `motion` is the default and sufficient. R3F/WebGL is NOT justified for the primary hero and should not be added unless a later stage demonstrates a concrete, gated benefit.**

Rationale (per the brief's 6-question test): the story is a data flow (claim -> three parallel sources -> one Gemini call -> sourced verdict). That reads best as a labeled, legible diagram, not a 3D object. WebGL would add bundle weight, GPU/mobile cost, SSR complexity, and accessibility burden for no comprehension gain. SVG/`motion` covers it with zero new deps, perfect crispness, and easy reduced-motion/offscreen handling.

`pipeline-visual.tsx` composition (mirrors README data flow and `how-it-works.tsx`):
```
[ Rumour / claim ]  ->  three converging lanes:
    • Civic knowledge base (25 facts)
    • RSS news index (17 NG newsrooms)
    • Tavily live search (16 NG domains)
              ->  [ Single Gemini 2.5 Flash call ]  ->  [ Verdict card: VERIFIED + evidence + Source ]
```
- Static-first: the full diagram is readable with no animation. Motion adds staged edge-draw (SVG `pathLength`) and a token pulse along lanes, looping only while in view and only when motion allowed.
- Reuse the real WhatsApp-bubble treatment already proven in `how-it-works.tsx` (green bar, verdict pill) for visual consistency.
- Must NOT depict authority routing / resolution tracking / civic-issue map as the core loop (those are `cs_website`/`fc_dashboard` features, not this bot). The map/report belong in the About cards as links to the sibling apps, not the hero pipeline.

### 4.1 Optional R3F path (only if a later stage justifies it, gated)
If ever added: `visual/civic-scene.tsx` + `visual/scene-fallback.tsx`, dynamically imported (`next/dynamic`, `ssr: false`), mounted only after `shouldUseEnhancedGraphics()` (reduced-motion off, `navigator.deviceMemory > 4` heuristic, WebGL available). `<Canvas dpr={[1,1.5]}>`, pause loop when offscreen (`useFrame` guarded by in-view), dispose on unmount. It would be a subtle background depth layer behind the diagram, never replacing it and never the LCP. Would require adding `three` + `@react-three/fiber` — a dependency add that must pass the FR-12.4 justification at that time. Absent that, no dependency is added.

## 5. Section-by-section plan

| Section | Keep | Redesign | Remove / fix |
|---|---|---|---|
| **Header** (`site-header.tsx`) | fixed bar, anchors, WhatsApp CTA, Radix mobile popover | scroll transparent->glass; active-section indicator via IntersectionObserver; use `GlassPanel` | move body-scroll lock into an effect w/ cleanup (currently mutates `document.body` during render); ensure focus mgmt |
| **Hero** (`hero-section.tsx`) | emerald "Vote informed" accent, WhatsApp CTA, tagline copy | eyebrow+headline+paragraph+dual CTA+real trust row; `next/image` slideshow (`hero-media.tsx`) with priority/lazy; add `pipeline-visual.tsx` | replace CSS `background-image` divs as LCP; reserve dimensions (CLS~0); add secondary CTA (View incident map) |
| **About** (`about-section.tsx`) | grid of feature cards, hover treatment; owner's kept copy | intro uses owner's authoritative two-paragraph copy (What is CivicSense / Why Nigeria needs this), split for layout; cards = live features (WhatsApp fact-check, Anonymous report+Map, RSS/Tavily grounding, Dashboard) shown normally + roadmap features (Election Calendar, Politician Watch) shown with a visible "Coming soon" badge; glass/token styling | reframe every "Target" metric as an aspiration ("Our goal:"/"Aiming for"), never achieved; "Coming soon" cards are non-links (no destination implying live) |
| **Sources** (`sources-ticker.tsx`) | 18 real logos via `next/image`, marquee, edge mask | pause on hover/focus + offscreen; reduced-motion static row; token borders | none functional; keep logos |
| **How it works** (`how-it-works.tsx`) | accurate 3-step, WhatsApp panels, reduced-motion-aware animated tab border | tokenize colors, use glass on the panel frame | keep content (it's accurate) |
| **Sample verdicts** (`sample-verdicts.tsx`) | repo-sourced verdict data (matches README), marquee | pause on hover/focus + offscreen; reduced-motion fallback; tokenize gradient | do not invent new verdicts |
| **Team** (`team-section.tsx`) | section shell, card layout | KEEP the section; replace all names with anonymized RANDOM/PLACEHOLDER display names + generic roles; add a code comment noting these are intentional anonymized placeholders | remove real handles (`adriel-babalola`, `debugAyo`) and real names ("David Adeola", "Promise Abiodu"); no icon linking a placeholder to a real profile |
| **CTA** (`call-to-action.tsx`) | starfield/grid parallax, real copy, dual CTA | pause parallax/starfield offscreen + reduced motion; fix mouse-position hook | fix `useEffect` missing deps + global listener leak in `use-relative-mouse-position.ts` |
| **Footer** (`site-footer.tsx`) | logo, nav, social | wire X/Instagram to real handles or drop them (currently both point to repo URL); tokenize | flag social placeholders |

No new sections are added purely for length. Every retained section has a purpose.

## 6. Animation strategy (concrete)
- Central `motion-tokens.ts` (durations/easings/variants). Components import from it.
- `useInViewPause(ref)` gates every `setInterval` (hero slideshow, sources & verdicts marquees), the CTA parallax `useScroll`/starfield loop, and the how-it-works animated border, plus the pipeline viz loop.
- `useReducedMotion()` (already used in hero + how-it-works) applied uniformly: reduced -> no loops, static frames, instant reveals.
- Hero entrance via `motion` variants with small `staggerChildren`.

## 7. Performance strategy
- **Images**: hero photos through `next/image` (AVIF/WebP auto, explicit width/height, `sizes`, `priority` only on LCP image, second lazy). Source logos already via `next/image`; keep. Reserve space to keep CLS~0.
- **Fonts**: Inter via `next/font` (already; self-hosted, no layout jump). Optionally add `display: "swap"`.
- **JS**: no new heavy deps by default. If R3F is ever added, it is dynamically imported and gated (§4.1) so it never ships to the initial bundle for unsupported/low-power clients.
- **Loops**: all paused offscreen and under reduced motion (§6). No rAF/interval/listener leaks (cleanup mandatory). Fix the CTA mouse hook leak.
- **Standalone build** (`output: "standalone"`) unaffected.

## 8. Accessibility strategy
- One `h1` (hero); sections use `h2`; cards use `h3`. `Section` primitive enforces this.
- `next/image` `alt` on meaningful media; decorative SVG/canvas `aria-hidden`, non-focusable.
- Visible focus rings everywhere (reuse existing emerald `focus-visible:ring` pattern from `action-button.tsx`/`ui/button.tsx`); never remove without replacement.
- AA contrast in all glass tiers and over hero scrim.
- Full keyboard path incl. Radix mobile menu (focus trap + restore, body scroll restored via effect cleanup).
- `prefers-reduced-motion` honored globally.

## 9. SEO strategy
- Preserve `metadata` structure in `layout.tsx` (title template, description from `siteConfig`, keywords, OpenGraph + Twitter with `opengraph-image.png`). Improve copy truthfully only.
- Flag `metadataBase: https://civicsense.app` and the placeholder `civicsense.app` links in `site-config.ts` for manual confirmation before publish; do not fabricate a canonical/live URL.
- Keep semantic heading order for crawlability.

## 10. Approach comparisons (decisions on record)
- **Pipeline viz: SVG/DOM vs R3F** -> **SVG/DOM.** Better legibility, zero deps, trivial reduced-motion/offscreen handling, no SSR/GPU cost. R3F deferred behind explicit gating and justification.
- **Glass: single primitive w/ tiers vs per-component ad-hoc blur** -> **single primitive.** Removes duplication, guarantees fallback + contrast, one place to tune.
- **Tokens: extend existing shadcn oklch layer vs replace** -> **extend.** Preserves `ui/*` components; adds semantic brand layer; lowest risk.
- **Roadmap features: remove vs advertise as "Coming soon"** -> **advertise as "Coming soon"** (owner decision, Stage B). Election Calendar and Politician Watch are real roadmap items; they stay on the page with a visible "Coming soon" badge and no live-availability claim. Building them is still out of scope.
- **Metrics: remove vs reframe as goals** -> **reframe as aspirations** (owner decision, Stage B). Numbers stay but are worded as targets ("Our goal:"), never achieved results.
- **Team: remove unverified names vs anonymize all** -> **anonymize all** (owner decision, Stage B). Section is kept; all names become random/placeholder for anonymity, with a code comment marking them as intentional placeholders. No real handles/names appear.
- **About copy: rewrite vs adopt owner copy** -> **adopt owner copy** as authoritative; split/format only, do not contradict.
- **Component folders: reorg vs minimal move** -> reorg is optional; prioritize primitives + copy + motion fixes over moving files. Only move if imports are updated and lint/typecheck/build stay green.

## 10a. Stage B hostile-review findings (issues + how the spec mitigates them)

Reviewed the revised spec against the actual code. Findings and the mitigations now baked in:

1. **"Coming soon" cards as dead links (a11y + truthfulness).** The current `about-section.tsx` wraps every card in `<a href target="_blank">`. A roadmap card pointing at `dashboardUrl` would imply the feature is live and would be a link to nowhere useful. Mitigation: FR-4.2/FR-4.4 require "Coming soon" cards to be rendered as non-anchor elements (e.g. a `<div>` / `<button disabled>`-style, or an `<a aria-disabled>` that does not navigate), with the "Coming soon" text in the accessible name (FR-4.5). Reuse the existing card map; do not build a separate component tree.
2. **Anonymized team + `Github` icon implying a real profile.** `team-section.tsx` uses a `Github` icon per member. With placeholder names, a Github icon (or any handle-style display) could imply a real account. Mitigation: FR-9.3 forbids linking placeholders to real profiles; prefer a neutral role icon (e.g. `User`/`Code`) over `Github`, and keep names visibly generic. This is a data-only change (swap the `TEAM` array + one comment); no structural rewrite.
3. **Metric reframing must not reintroduce fabricated precision.** Rewording "10M+ youths" as "Our goal: reach 10M+ youths" is truthful; inventing a NEW precise number is not. Mitigation: FR-4.3 keeps only the owner's existing numbers, reworded as aims; no new figures.
4. **No new dependencies — confirmed.** `package.json` has no `three`/`@react-three/fiber`; `motion`, Radix, lucide, cva/clsx/tailwind-merge, vaul cover everything. The plan adds zero deps (FR-12.4). The optional R3F path (§4.1) remains explicitly deferred and gated; it is NOT part of Stage C.
5. **Scope creep in copy tasks.** Tasks 8 and 10 are data/copy edits over existing structures (card array, team array, intro paragraphs) plus token/glass styling — they must NOT trigger a full component rewrite. Mitigation: Task 8/10 wording now says reuse the existing map/structure.
6. **Footer social links point to the repo URL (X + Instagram).** Confirmed in `site-footer.tsx` (both `Icons.x` and `Icons.instagram` link to `repositoryUrl`). Mitigation unchanged from Stage A (Task 10): wire to real handles or remove; do not present a repo link as a social profile. Flag for manual confirmation (FR-11) since no real handles are in the repo.
7. **Mouse-position hook leak — confirmed.** `use-relative-mouse-position.ts` `useEffect` has no dependency array, so it re-registers a global `mousemove` listener every render. Mitigation unchanged (Task 2 / FR-8.2): add deps, scope to hover/in-view, clean up.
8. **Reduced-motion / offscreen gating unchanged and sufficient.** No new animated surfaces are introduced by the Stage B copy changes; the "Coming soon" badge is static styling, so it adds no motion/perf risk.

## 11. Risks & mitigations
- `next/image` on hero changing layout -> reserve dimensions, verify CLS in visual QA.
- `backdrop-filter` inconsistency -> `@supports` fallback tier C.
- Marquee pause logic causing jump -> pause by freezing `motion` animation (playbackRate/`animationPlayState`), not unmounting.
- Fixing `use-relative-mouse-position` deps could change behavior -> scope listener to CTA hover/in-view; verify no regression.
- Folder moves risking broken `@/` imports -> keep moves minimal; run typecheck/build after any move.
- No test suite exists -> rely on lint + typecheck + build + manual visual QA (dev server) as the gates.
