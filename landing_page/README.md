# CivicSense — Landing Page

The public landing page for **CivicSense**, a WhatsApp AI fact-checking bot for Nigerians.

> Send a rumour to WhatsApp. Get the truth back.

Built with Next.js 16, React 19, TailwindCSS v4, Shadcn UI and Motion.

## Sections

- **Hero** — animated cosmic planet with the CivicSense wordmark and the WhatsApp CTA
- **Sources** — marquee of the 17 Nigerian newsrooms and IFCN fact-checkers the bot verifies against
- **How it works** — three interactive steps from rumour to a sourced verdict
- **Sample verdicts** — scrolling cards of real claims from the civic knowledge base
- **Call to action** — the WhatsApp invite and anonymous report entry point

## Run it

```bash
npm install
npm run dev        # http://localhost:3000
npm run build      # production build
npm run typecheck  # tsc --noEmit
npm run lint       # biome check .
```

## Notes

- The WhatsApp CTA opens `https://wa.me/14155238886` (Twilio sandbox). Update
  `src/config/site-config.ts` when the production number is ready.
- `mapUrl`, `reportUrl` and `dashboardUrl` in `src/config/site-config.ts` are
  placeholders — replace them with the live `cs_website` / `fc_dashboard`
  deployments.
- News outlet favicons in `src/assets/sources/` are the official site icons,
  fetched from each outlet's own domain.