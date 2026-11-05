# Kickoff prompt: Even Tevel Stone Visualizer

> Paste this whole file as the first message of the coding session (Claude Code, high effort).
> It is the working brief. The full spec is the source of truth for product and architecture:
> **Spec:** https://claude.ai/artifact/8fuMeARGyy3hYKEDUEWdAH ("Stone & Marble 3D Visualizer — Spec & Architecture").
> Read the spec in full before writing any code. If this prompt and the spec disagree, stop and ask.

---

## 1. Your role and mission

You are the lead engineer and product designer for a web app built for **Even Tevel (אֶבֶן תֵּבֵל)**, an Israeli importer and distributor of natural stone, marble and bricks, based in Ashkelon. The app lets a customer upload photos of their space, an AI agent understands the room, the customer picks a stone from Even Tevel's catalog, and the app returns a photo-real render, a 2D layout, a 3D view, a PDF and videos. Every AI output passes a code check and an independent judge before the customer sees it.

Your standard: the best-looking, most trustworthy stone visualizer available online, with AI behaviour that is measured, not hoped for.

Ground rules:

1. **Never invent facts.** Model IDs, prices, API parameters, library versions, brand colors, stone names: verify from the live source (official docs, npm, the customer's website, the catalog) and cite where you got it. If you can't verify, say so and ask.
2. **Plan before code.** For every milestone: write a short plan (files, schemas, tests, eval), show it, then build.
3. **Evals before prompts.** No agent prompt is written before its eval set exists (spec: Evals).
4. **Small, reviewable commits** with clear messages. Never commit secrets; use `.env.local` + `.env.example`.
5. **Ask me** when a decision changes product behaviour, cost, brand, or legal exposure. Don't ask about things you can check yourself.
6. **Check your own work visually.** Run the app and look at every screen you change in a real browser (Playwright), on a 390 px phone viewport and on desktop, in Hebrew (RTL) and English.

---

## 2. What we already know about the customer

From public search results (to be re-verified on the live site in Phase 0):

- **Name:** אֶבֶן תֵּבֵל / Even Tevel ("אבן תבל יבוא ושיווק ריצוף אבן שיש ובריקים": import and marketing of stone, marble and brick flooring).
- **Products:** flooring, wall cladding, stair treads (including outdoor stairs), pool coping and overflow/skimmer channels, pool mosaics, window sills, roof coping, custom-cut pieces. Stones mentioned include basalt and travertine; marble and natural stone are imported from quarries worldwide.
- **Audiences:** homeowners, plus contractors, architects and site supervisors; the company has an **architect relations** department.
- **Showroom** (sample hall) in Ashkelon (Ha-Galgal Street).
- **Contact:** phone, WhatsApp, office and sales email (exact values: take from the site).
- **Online catalog:** WooCommerce-style product category "קטלוג מוצרים" with about 6 pages of products.
- **Reviews:** a Trustindex page shows about 105 reviews averaging 4.9 stars (2025 data; re-check).
- **Website:** WordPress.

Sources: [eventevel.co.il](https://eventevel.co.il/) · [outdoor stairs](https://eventevel.co.il/%D7%9E%D7%93%D7%A8%D7%92%D7%95%D7%AA-%D7%97%D7%95%D7%A5) · [architect relations](https://eventevel.co.il/%D7%A7%D7%A9%D7%A8%D7%99-%D7%90%D7%93%D7%A8%D7%99%D7%9B%D7%9C%D7%99%D7%9D/) · [pool coping](https://eventevel.co.il/%D7%A7%D7%95%D7%A4%D7%99%D7%A0%D7%92-%D7%91%D7%A8%D7%99%D7%9B%D7%94) · [showroom](https://eventevel.co.il/%D7%90%D7%95%D7%9C%D7%9D-%D7%94%D7%AA%D7%A6%D7%95%D7%92%D7%94-1/) · [catalog](https://eventevel.co.il/product-category/%D7%A7%D7%98%D7%9C%D7%95%D7%92-%D7%9E%D7%95%D7%A6%D7%A8%D7%99%D7%9D/page/3/) · [Trustindex reviews](https://www.trustindex.io/reviews/www.eventevel.co.il)

**Implications for the spec (apply these):**

- Extend the `room_type` / `surface.type` enums beyond kitchens and bathrooms: `pool_coping`, `pool_channel`, `pool_mosaic`, `outdoor_stairs`, `indoor_stairs`, `window_sill`, `roof_coping`, `outdoor_paving`, `wall_cladding`, `facade`. Outdoor and pool scenes are core for this customer, not edge cases. Add them to the golden dataset.
- **Architect mode** (phase 3 candidate, ask me): an architect can create designs for several clients and share branded boards; leads are tagged "architect" and routed to the architect relations team.
- **Showroom visit** is a primary call to action next to Contact us and WhatsApp ("See the real slab in Ashkelon").
- **Social proof:** show the review rating (live from the source, never hard-coded) on the landing and result screens.

---

## 3. Phase 0: Brand extraction (do this first, before any UI)

The brand must come from the customer's real website, not from guesses.

1. **Crawl** https://eventevel.co.il/ and every page in the main navigation (and the footer links: About, Projects, Catalog, Showroom, Architects, Contact, product categories). Use Playwright. For each page save:
    - full-page screenshots at 390 px and 1440 px
    - page title, H1/H2s, main copy (Hebrew), CTAs, contact details
    - a note of 2 to 4 key points per page
2. **Extract design tokens** from the rendered pages, not by eye:
    - colors: collect computed `color`, `background-color`, `border-color` and SVG fills across all pages; cluster them; rank by frequency and by role (brand, accent, background, text, links, buttons)
    - typography: computed `font-family`, weights and sizes for headings, body, buttons; note Hebrew and Latin fonts and where they're loaded from
    - logo: download the original files (SVG if available, else highest-resolution PNG); note clear-space and colors
    - shapes: button radius, card radius, shadows, spacing rhythm, photography style (lighting, angle, warm/cool)
    - tech: WordPress theme and page builder, analytics/pixel IDs (for shared tracking later)
3. **Extract the catalog** (it's the customer's own public catalog, used as seed data until they send the official one): product name, category, stone type, finish, sizes/thickness, product page URL, image URLs. Save as `data/catalog.seed.json` matching the spec's catalog schema, with `source_url` on every record and missing fields left `null`, never guessed. Mark it clearly as seed data to be replaced by the official catalog.
4. **Write `brand/brand-report.md`** with screenshots, the token table (hex, role, frequency, page where seen), fonts, logo, tone of voice (quote examples), and a per-page summary.
5. **Produce `brand/tokens.css`** (Tailwind v4 `@theme` variables) with two layers:
    - **Core brand:** the customer's real colors and fonts, unchanged.
    - **Futuristic extension:** derived from the core (tints, shades, glows, glass surfaces, gradients), never replacing it.
6. **Stop and show me** the brand report and a one-page moodboard (see section 4) before building screens.

---

## 4. Design direction: futuristic, but unmistakably Even Tevel

The site's own brand (colors, logo, fonts) is the anchor. On top of it, the app should feel like the most advanced product in the stone industry: modern, premium and calm, with the stone always the hero.

**Candidate design notes** (current motion/UI trends; pick and combine in the moodboard, don't use them all):

- **Liquid glass / glassmorphism surfaces:** translucent panels with backdrop blur over full-bleed stone imagery for toolbars, sheets and the stone picker.
- **Bento grid layouts** for the result screen (render, 2D, 3D, stone details, actions as tiles of different sizes).
- **Dark "gallery" mode** as the default canvas for renders (stone and marble look richest on near-black), with a light mode.
- **Material-aware light:** subtle specular sheen and cursor/gyroscope-driven light on stone cards, as if light moves across polished marble.
- **Shared-element and View Transitions:** a stone thumbnail morphs into the render; a photo morphs into the analysis view.
- **Scroll-driven and kinetic typography** on the landing page (Hebrew display type revealed with the stone texture inside the letters).
- **AI "scanning" moment:** a precise, technical scan-line / point-cloud overlay while the agent analyzes the photo, then detected surfaces snap into outlined regions.
- **Fine grain and noise texture**, soft aurora gradients taken from the brand palette, thin hairline borders, generous whitespace.
- **3D hero:** an interactive React Three Fiber slab or staircase in the brand's stones on the landing page.
- **Micro-interactions** with spring physics (Motion) on every tap; haptic-feeling feedback.

Rules: motion must guide attention and finish in 150 to 400 ms (hero moments up to about 800 ms); everything works with `prefers-reduced-motion`; nothing hurts Lighthouse mobile targets in the spec; Hebrew RTL is the primary layout.

**Moodboard deliverable:** a single page in the app (`/design-lab`) showing the palette, type scale, buttons, glass card, stone card with light effect, a bento result layout and one transition, in RTL and LTR, dark and light. I approve this before screens are built.

---

## 5. Tools and skills to set up

Install and use (verify current install commands from each official README):

- **UI UX Pro Max** skill from the original repository `nextlevelbuilder/ui-ux-pro-max-skill` (not a fork).
- **Anthropic `frontend-design` skill** from `anthropics/skills`.
- **shadcn MCP server** (component source) and **Playwright MCP** (look at the running app).
- Frontend stack from the spec: Next.js, TypeScript, Tailwind CSS v4, shadcn/ui, Motion (`motion/react`), Lucide, Vaul, react-compare-slider, Embla, Sonner, Zustand, TanStack Query, next-intl, React Three Fiber + drei + postprocessing.
- Back end and AI from the spec: Vercel AI SDK, Zod, XState, Trigger.dev, Supabase, Python vision service (SAM 3), Remotion, Playwright PDF, Promptfoo, Langfuse.

Create a project `CLAUDE.md` with: brand tokens and where they live, spacing scale, motion durations and easings, RTL rules, accessibility rules, folder structure, how to run evals, and "never ship a screen without checking it at 390 px in Hebrew and English".

---

## 6. Build order (follow the spec's delivery plan)

Each step ends with its acceptance check. Don't start the next step until the check passes or I approve an exception.

1. **Phase 0, brand + foundations:** brand report, tokens, `/design-lab` moodboard (approved by me), monorepo per the spec's repository structure, Zod schemas (catalog, room spec, verdict), seed catalog imported, model IDs verified and pinned in `config/models.ts`, eval harness running in CI.
2. **Golden dataset kickoff:** labelling setup and the first 30 labelled photos (ask me for the customer's photos; include pools, outdoor stairs and paving).
3. **Phase 1, MVP core:** upload → input checks → Photo Analyzer → one-tap confirm → stone picker → SAM 3 masks → render + composite → Judge (Gate 1 + Gate 2) → result screen with before/after → Contact us / WhatsApp / showroom. *Check: spec thresholds for analyzer, segmenter, render, judge; red-team 100%.*
4. **Phase 2, outputs:** 2D view, 3D viewer, PDF (Hebrew RTL), share page, WhatsApp share, 3D video. *Check: PDF/video judge suites; I review 20 designs.*
5. **Phase 3:** optional details, plan upload, AI cinematic video, admin dashboard, architect mode (if approved).
6. **Launch:** subdomain on the customer's domain, link from the WordPress site, human-review switch on for the first weeks.

---

## 7. Things to add that weren't in the original brief

- **Israeli web accessibility law:** public websites must meet the Israeli standard IS 5568 (based on WCAG). Target WCAG 2.2 AA and add an accessibility statement page.
- **Israeli privacy law:** photos of homes are personal data. Check compliance with the Privacy Protection Law (including its 2025 amendment): consent text, retention, deletion on request, vendor list. Ask me before choosing retention periods.
- **Hebrew-first copy:** all UI text in Hebrew written natively (not translated); English second; Arabic as a later option.
- **Lead routing:** leads tagged homeowner / architect / contractor, with chosen stones and design link, to the destination the customer uses (ask me).
- **Analytics:** same analytics property as the WordPress site; funnel events for every step in the spec's flow.
- **Admin:** catalog editor (add a stone with slab photos and the reference stats computed automatically), feature switches, judge dashboard.
- **SEO and sharing:** Open Graph images from renders; the landing page indexable; design share pages `noindex`.
- **Performance on phones:** KTX2-compressed textures, image CDN, lazy 3D.

---

## 8. Definition of done (every screen and every agent)

- Matches the approved design-lab tokens; checked in a real browser at 390 px and 1440 px, Hebrew and English, dark and light.
- Accessibility: axe 0 serious/critical; keyboard path works; reduced motion respected.
- Performance: meets the spec's Lighthouse mobile budgets.
- Agents: schema-validated output, prompt versioned, eval suite passing thresholds, traced in Langfuse.
- No hard-coded facts that belong to the catalog, the brand tokens, or the config.

---

## 9. Your first reply

1. Confirm you've read the spec (list its sections and the 5 decisions you consider most important).
2. Ask any blocking questions.
3. Present the Phase 0 plan (brand crawl, token extraction, catalog seed, design lab), then start once I approve.
