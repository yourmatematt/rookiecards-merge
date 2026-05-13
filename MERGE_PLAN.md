# RookieCards Merge Plan

**Author:** Claude (analysis pass — no codebase modifications)
**Date:** 2026-05-13
**Repos analysed:**
- `junior-hero-moments` (Lovable export — new frontend design)
- `rookiecards` (current live app — backend and integrations)

---

## 1. Clone & Build Status

Both repos cloned into `C:\projects\rookiecards-merge\`.

| Repo | npm install | npm run build |
|---|---|---|
| `junior-hero-moments` | ✅ `added 513 packages in ~1m` | ✅ Vite 7 client + SSR build, `built in 2.40s` / `2.19s` |
| `rookiecards` (monorepo) | ✅ `added 541 packages in 11s` | ✅ Vite 5 client `built in 2.42s` + `tsc` server, no errors |

Both repos build cleanly on this Windows machine. Native deps in `rookiecards/server` (`better-sqlite3`, `sharp`, `@imgly/background-removal-node`) installed without prebuilt-binary or node-gyp issues.

Build warnings noted (non-blocking):
- `rookiecards/client`: main bundle 538 kB — chunk-size warning, suggests dynamic imports.
- `junior-hero-moments`: SSR server bundle 727 kB — Cloudflare Workers compatibility-flag `nodejs_compat` is on, so this is expected.

---

## 2. Inventory

### 2A · `junior-hero-moments` (Lovable export)

#### STACK
- **Framework:** TanStack Start `^1.167.50` (file-based routing via `@tanstack/react-router` + `@tanstack/router-plugin`), Vite `^7.3.1`, deployed to **Cloudflare Workers** (`wrangler.jsonc`, `@cloudflare/vite-plugin`).
- **React:** **19.2** (with `react-dom@19.2`).
- **TypeScript:** `5.8.3`, strict mode, `"@/*"` path alias.
- **Styling:** **Tailwind 4** (`@tailwindcss/vite` plugin — no `tailwind.config.js`, all config in `styles.css` via `@theme`).
- **Routing:** `@tanstack/react-router` with filesystem routes under `src/routes/` (route tree auto-generated → `routeTree.gen.ts`).
- **State / forms:** `@tanstack/react-query` (provided but barely used), `react-hook-form` + `zod` + `@hookform/resolvers`.
- **UI library:** Full **shadcn/ui** (style: `new-york`, base: `slate`) — 46 Radix-based components in `src/components/ui/`. `components.json` present.

#### DEPENDENCIES (production)
- **UI / components:** all `@radix-ui/react-*` primitives (28 of them), `lucide-react@^0.575`, `cmdk`, `vaul`, `sonner`, `embla-carousel-react`, `recharts`, `framer-motion@^12.38`, `input-otp`, `react-day-picker`, `react-resizable-panels`, `class-variance-authority`, `clsx`, `tailwind-merge`, `tw-animate-css`.
- **Backend SDKs:** **None.**
- **Payment libs:** **None.**
- **Email libs:** **None.**
- **Auth libs:** **None.**
- **ORM / DB:** **None.**
- **Forms / validation:** `react-hook-form@^7.71`, `zod@^3.24`.
- **Build-critical dev:** `@lovable.dev/vite-tanstack-config`, `@cloudflare/vite-plugin`.

#### STRUCTURE (`src/`, 2 levels)
```
src/
├── assets/              (5 JPGs: hero-kid, box, three-tiers, team, proud-kid)
├── components/
│   ├── site/            (Header, Footer, SlabCard — 3 files)
│   └── ui/              (46 shadcn primitives)
├── hooks/               (use-mobile.tsx)
├── lib/
│   ├── error-capture.ts
│   ├── error-page.ts
│   ├── site.ts          (constants: FOUNDING_TOTAL=75, FOUNDING_CLAIMED=28, PRICE_AUD=34.95)
│   └── utils.ts         (cn() helper)
├── routes/              (file-based — 6 files)
├── routeTree.gen.ts     (auto-generated)
├── router.tsx
├── server.ts            (Cloudflare Workers SSR entry)
├── start.ts
└── styles.css
```

#### ROUTES (6 total)
| Path | File | Purpose | Backend wired? |
|---|---|---|---|
| `/` | `routes/index.tsx` (348 lines) | Hero + FOUNDING 75 mechanic + tiers + how it works + FAQ | ❌ (visual only) |
| `/order` | `routes/order.tsx` (215 lines) | 5-step order wizard | ❌ `setDone(true); toast.success("Order received!")` |
| `/for-clubs` | `routes/for-clubs.tsx` (125 lines) | Club partner registration form | ❌ `setSubmitted(true); toast.success()` |
| `/how-it-works` | `routes/how-it-works.tsx` (68 lines) | Static explainer | n/a |
| `/contact` | `routes/contact.tsx` (72 lines) | Contact form | ❌ visual only |
| `__root` | `routes/__root.tsx` (121 lines) | Root shell, meta, 404 + error components | n/a |

#### BACKEND INTEGRATIONS
- **None.** Zero env-var references in `src/`. No `import.meta.env.*`, no `process.env.*`, no `.env.example`. No API fetches anywhere. All form submits resolve to `toast.success()` and a local `setState`.
- Cloudflare Workers handler (`src/server.ts`) is the TanStack Start SSR wrapper only — no server logic.

#### ENV VARS EXPECTED
- None.

---

### 2B · `rookiecards` (live app)

#### STACK
- **Monorepo:** `client/` + `server/` workspaces via root `package.json`. Top-level dev: `concurrently`.
- **Client framework:** Vite `^5.1.6` + React `^18.2` SPA (deployed to **Cloudflare Pages**, see `client/cloudflare-pages.json`). Custom domain `rookiecards.com.au`.
- **Server framework:** Express `^4.18.3` on Node 20+ (deployed to **Railway**, see `server/railway.json`, NIXPACKS builder, health check `/health`).
- **TypeScript:** `5.4.2` both sides.
- **Styling:** **Tailwind 3** (`client/tailwind.config.js` — JS config, theme extended with brand colours `rookieBlue/rookieGold/rookieSilver`, custom keyframes for floats/shimmer/pulse-glow), `@tailwindcss/typography` plugin. No shadcn/ui registry — bespoke components.
- **Routing:** `react-router-dom@^6.22` with declarative `<Routes>`.
- **State / forms:** Local component state only. No Zustand/Redux/React Query/react-hook-form. Custom typed fetch wrappers in `client/src/lib/api.ts`.
- **MDX:** `@mdx-js/rollup` + `@mdx-js/react` + remark plugins → guides system (content lives in `/content/guides`, rendered by `pages/GuideArticle.tsx`).

#### DEPENDENCIES (client production)
- `react@^18.2`, `react-dom@^18.2`, `react-router-dom@^6.22`, `framer-motion@^12.38`, `lucide-react@^1.7.0` ⚠ (unusual major-version pin — likely typo for `^0.x`; effectively pre-2024), `browser-image-compression@^2.0.2`.
- Build-critical dev: `@mdx-js/rollup`, `@mdx-js/react`, `@tailwindcss/typography`, `tailwindcss@^3.4`, `@vitejs/plugin-react@^4.2`, `gray-matter`, `remark-frontmatter`, `remark-mdx-frontmatter`, `remark-gfm`, `@resvg/resvg-js` + `jpeg-js` (for `scripts/render-guide-heroes.mjs`).

#### DEPENDENCIES (server production)
- **Backend / runtime:** `express`, `cors`, `dotenv`, `multer` (uploads), `better-sqlite3@^9.4` (DB), `node-fetch@^2`, `node-cron` (cleanup jobs), `form-data`, `uuid`.
- **Payments:** `stripe@^14.21`.
- **Email:** `resend@^6.12`, `nodemailer@^8.0` (+ `@types/nodemailer`).
- **Image / AI:** `sharp@^0.33`, `@imgly/background-removal-node@^1.4.5`.

#### STRUCTURE (2 levels)
```
rookiecards/
├── brand/, brand-assets/
├── content/
│   └── guides/                 (MDX articles)
├── design-system/, docs/, marketing/, outputs/
├── pipeline/                   (legacy n8n workflows — see CLAUDE.md inside)
├── client/
│   ├── index.html, postcss.config.js, tailwind.config.js
│   ├── cloudflare-pages.json   (deploy config)
│   ├── public/, scripts/       (generate-feeds, render-guide-heroes)
│   ├── vite.config.ts          (raw-MDX plugin + MDX + feeds plugin + /api proxy)
│   └── src/
│       ├── App.tsx, main.tsx, index.css
│       ├── components/         (10 components + guides/MdxComponents+Seo)
│       ├── lib/                (api.ts, compress.ts, guides.ts)
│       └── pages/              (11 pages — see ROUTES below)
└── server/
    ├── railway.json
    ├── .env.example            (16 vars — listed below)
    └── src/
        ├── index.ts            (Express bootstrap, CORS, route mount, cleanup jobs)
        ├── composite-pipeline/ (NEW AI flow: backgrounds, bgRemoval, compositor/, frame*, rateLimiter, storage, watermark)
        ├── db/                 (schema.ts — 8 tables w/ migrations; seed.ts)
        ├── jobs/               (photoCleanup, previewCleanup — node-cron)
        ├── lib/                (email.ts — 8 Resend templates; image.ts; pipeline.ts — legacy n8n trigger)
        └── routes/             (10 routers — see BACKEND below)
```

#### ROUTES — client (11 pages, defined in `client/src/App.tsx`)
| Path | Component | Lines |
|---|---|---|
| `/` | `ClubLanding` | 767 |
| `/order/:clubSlug` | `OrderFlow` | **1042** |
| `/confirmation/:orderId` | `Confirmation` | 408 |
| `/share/:token` | `SharePage` | 229 |
| `/admin` | `AdminDashboard` | 367 |
| `/interest` | `InterestForm` | 520 |
| `/contact` | `ContactPage` | 304 |
| `/preview` | `ParentLanding` | 1011 |
| `/for/:clubSlug` | `ClubParentLanding` | 574 |
| `/guides`, `/guides/:slug` | `GuidesIndex`, `GuideArticle` (lazy + MDX) | 235 + 348 |
| `*` | `NotFound` | 25 |
| **Total client page LOC** | | **~6 760** |

#### BACKEND — server (10 routers under `/api/*`)
| Mount | File | Responsibilities |
|---|---|---|
| `/api/clubs` | `clubs.ts` | Club lookup by slug |
| `/api/orders` | `orders.ts` | Order create (multer photo upload, SQLite insert) → Stripe Checkout Session create → confirmation fetch |
| `/api/webhooks/stripe` | `webhooks.ts` | Stripe signature verification, mark paid, trigger pipeline, fire Resend confirmation emails |
| `/api/admin` | `admin.ts` | Admin dashboard data (gated by `ADMIN_API_KEY`) |
| `/api/callbacks` | `callbacks.ts` | n8n / pipeline callbacks (fronts_generated etc.), writes to `card-outputs/` |
| `/api/interests` | `interests.ts` | Parent + club registration capture (`parent_interests` table) |
| `/api/share/:token` | `share.ts` | Public share view of generated cards |
| `/api/trophy-orders` | `trophyOrders.ts` | Bulk presentation-night order flow + Stripe |
| `/api/contact` | `contact.ts` | Contact form → admin email via Resend |
| `/api/composite/*` + `/api/backgrounds` | `composite.ts` | **New gen pipeline** — generates watermarked preview, lists backgrounds; Phase 2 finalise endpoint reserved |
| Static: `/card-outputs`, `/backgrounds`, `/preview-outputs` | (express.static) | UUID-scoped output serving |

**Database (better-sqlite3 — WAL mode, FKs on):**
- `clubs` (id, slug, name, sport, logo_url, colours, deadline, pickup_details)
- `orders` (status pending→paid→processing→complete; pack_quantity, total_cents, source, promo_code, email, share_token, athlete_data JSON, card_mode, display_stand)
- `recipients` (per-order, ≤10, gift_message)
- `consent_records` (text + IP)
- `card_generations` (per-style, preview/print URLs)
- `parent_interests` (lead capture, dedupe on email+club_slug, utm tracking)
- `previews` (new flow: watermarked preview JPGs, compositor cost, IP, 7-day TTL)
- `backgrounds` (curated bg library, variant: standard/silver/gold, designed flag)

**Cron jobs:** `photoCleanup` (90-day delete uploaded photos), `previewCleanup` (purge expired previews).

#### EMAIL FLOWS (Resend, `server/src/lib/email.ts`)
8 transactional templates — all branded RookieCards (navy `#0B1D3A` + gold `#C9A84C`):
1. `sendAcquisitionInterestConfirmationEmail` — club coordinator
2. `sendInterestConfirmationEmail` — parent who registered interest
3. `sendOrderConfirmationEmail` — post-payment
4. `sendAbandonedCartEmail` — preview-pipeline re-engagement
5. `sendTrophyOrderConfirmationEmail` — bulk presentation orders
6. `sendClubActivationEmail` — broadcast when a club goes live
7. `sendContactFormEmail` — admin alert (with Reply-To to sender)
8. `sendNewClubRegistrationAlert` — admin alert on new club registration

#### ENV VARS EXPECTED (`server/.env.example` + grep)
| Var | Required | Purpose |
|---|---|---|
| `PORT` | optional (3001) | Express port |
| `CLIENT_URL` | yes | comma-separated CORS allowlist; first entry used for Stripe redirect + email links |
| `STRIPE_SECRET_KEY` | yes | Stripe API |
| `STRIPE_WEBHOOK_SECRET` | yes | Webhook sig verification |
| `STRIPE_PUBLISHABLE_KEY` | (client) | Currently not wired client-side — uses Checkout |
| `ADMIN_API_KEY` | yes | Admin dashboard gate |
| `DATABASE_PATH` | optional | SQLite path |
| `UPLOADS_DIR` | optional | Photo storage |
| `MAX_UPLOADS_MB` | optional (1024) | Cleanup threshold |
| `RESEND_API_KEY` | optional (silently no-ops) | Email |
| `EMAIL_FROM` | optional | `RookieCards <rookiecards@yourmateagency.com.au>` |
| `N8N_WEBHOOK_BASE_URL` | optional | Legacy pipeline trigger |
| `SERVER_URL` | yes | Public base for n8n callbacks + static asset URLs |
| `REPLICATE_API_TOKEN` | yes (for new flow) | Flux Kontext AI compositor |
| `COMPOSITOR_PROVIDER`, `COMPOSITOR_MODEL` | optional | Defaults `replicate-flux-kontext` / `flux-kontext-pro` |
| `PREVIEW_OUTPUTS_DIR`, `PRINT_OUTPUTS_DIR`, `BACKGROUNDS_DIR`, `FRAMES_DIR` | optional | Local storage paths |

Client side: `VITE_API_URL` (defaults to `/api` via Vite proxy in dev; Cloudflare Pages prod sets `https://server-production-3de3.up.railway.app/api`).

#### THINGS THE TASK ASKED ME TO FLAG IN ROOKIECARDS
| Asked for | Found? | Notes |
|---|---|---|
| Existing order management | ✅ | Full pipeline: photo upload → SQLite order → Stripe Checkout → webhook → AI gen → emails → share link |
| Existing payment integration | ✅ | Stripe Checkout Sessions (AUD), webhooks with signature verification, raw-body parsing on `/api/webhooks`, both rookie packs and trophy orders |
| Existing club registration flow | ✅ | `InterestForm` (520 LOC) + `/api/interests` + dedupe + admin alert email + parent confirmation email |
| Existing photo upload handling | ✅ | Multer disk storage, 5 MB limit, UUID filename, client-side `browser-image-compression` pre-upload, 90-day cron deletion, consent record with IP |
| Existing customer email flows | ✅ | 8 Resend templates (see above) |
| Founding 75 launch mechanic, refund-for-UGC, $500 prize comp | ❌ | **Not present in rookiecards code.** Only mention of "Founding 75" appears in `junior-hero-moments` (constants in `src/lib/site.ts`, used in the index hero). Rookiecards has a **different** mechanic: "Charter Partner Clubs" — 10% kickback in club gear, first-50 clubs framing (in `client/src/pages/ClubLanding.tsx` copy and `sendAcquisitionInterestConfirmationEmail`). No refund-for-UGC code, no $500 prize comp code anywhere. |

---

## 3. Compare & Contrast

### Overlap (same feature, different implementation)
| Feature | `junior-hero-moments` | `rookiecards` |
|---|---|---|
| Home / hero | `routes/index.tsx` (348 LOC, FOUNDING 75, gold gradient, Anton display, framer-motion) | `pages/ClubLanding.tsx` (767 LOC, Charter Partner copy, navy/gold, custom keyframes) |
| Order wizard | `routes/order.tsx` (215 LOC — mock, 5 steps, no API) | `pages/OrderFlow.tsx` (**1042** LOC — real, branches on ad-traffic, compresses photos, posts FormData, redirects to Stripe) |
| Club partner page | `routes/for-clubs.tsx` (125 LOC — `setSubmitted(true)`) | `pages/InterestForm.tsx` (520 LOC — real submit, dedupe, sends both confirmation + admin emails) |
| Contact | `routes/contact.tsx` (72 LOC — mock) | `pages/ContactPage.tsx` (304 LOC — real POST + admin email) |
| How-it-works | `routes/how-it-works.tsx` (68 LOC) | Embedded in `ClubLanding`, `ParentLanding`, etc. |

### Only in `junior-hero-moments`
- A full **shadcn/ui** component library (46 primitives, `components.json` registered).
- **Tailwind 4 + CSS vars** styling system (different from rookiecards' Tailwind 3 + JS config).
- **TanStack Start** filesystem routing on **React 19**, deployed via **Cloudflare Workers**.
- A polished visual language: Anton display font, slate base palette w/ gold accents, framer-motion micro-animations, FOUNDING-75 mechanic UI.
- Five hero photographs (`src/assets/*.jpg`) chosen for the new look.

### Only in `rookiecards`
- **Everything backend.** Express + SQLite + Stripe + Resend + AI image generation pipeline + cron jobs + admin dashboard + share-link mechanic + trophy-card flow + composite-preview pipeline + n8n integration.
- **MDX guides system** (content in `/content/guides`, build-time feed generation, schema.org SEO).
- **Production deployment** (Cloudflare Pages → Railway), live custom domain (`rookiecards.com.au`).
- **Real user flows** with 6 760+ LOC of working page code, branded transactional emails, payment-verified status transitions.
- **Brand & marketing assets** (`brand/`, `brand-assets/`, `marketing/`, `design-system/`, `docs/`, `outputs/`).
- **Charter Partner / 10% kickback** business mechanic (not a launch lottery — an ongoing club-recruitment lever).

### Stack compatibility — **incompatible without migration work**
| Concern | JHM | Rookiecards | Compatible? |
|---|---|---|---|
| React | **19.2** | 18.2 | ⚠ React 19 has a few breaking changes (Suspense, ref-as-prop, defaultProps on FCs) — upgrading rookiecards client is doable but introduces risk. |
| Router | **TanStack Router** (filesystem) | **react-router-dom** v6 | ❌ Fundamentally different — cannot mix. Route files don't port directly; they must be rewritten as `<Route>` declarations (or vice versa). |
| Bundler | **Vite 7** (+ `@cloudflare/vite-plugin` + TanStack plugin) | **Vite 5** (+ custom raw-MDX plugin + `@mdx-js/rollup`) | ⚠ Different Vite majors. MDX pipeline is custom and load-bearing for guides; cannot be lost. |
| CSS | **Tailwind 4** (CSS-first config in `styles.css`) | **Tailwind 3** (JS config + `@tailwindcss/typography`) | ⚠ Tailwind 4 is a major upgrade. Brand-colour tokens differ. Typography plugin is required for MDX guides. |
| Icon library | `lucide-react@^0.575` | `lucide-react@^1.7.0` (suspicious major) | ⚠ Version mismatch; APIs and icon names differ. |
| Component library | Full **shadcn/ui** (46 Radix primitives) | Bespoke components, no Radix at all | New addition either way. |
| Forms | `react-hook-form` + `zod` | Plain `useState` | New addition either way. |
| Deployment | Cloudflare **Workers** (SSR) | Cloudflare **Pages** (SPA) + Railway server | ⚠ Different Cloudflare product. SSR vs SPA. Migrating to Workers would break Railway-backed server expectations. |

---

## 4. Proposed Merge Strategy — **Option A (refined): rookiecards as base, JHM as design source**

### Recommendation
**Use `rookiecards` as the base.** Port the **visual language** of `junior-hero-moments` on top — but treat JHM's code as a *reference* for design tokens, component library, and page-level mockups, not as code to import wholesale.

### Why not the other options
- **B (JHM base, port rookiecards integrations on top)** — would require rebuilding 6 760 LOC of working pages, the entire 10-router Express API, 8 Resend templates, the SQLite schema with 8 tables and live migrations, the composite-preview pipeline with Replicate, the AdminDashboard, the share-link mechanic, the MDX guides system, the trophy-order flow, and the CI/CD across Cloudflare Pages + Railway. This is **weeks** of work and would walk live customers off a working app onto an unproven SSR-on-Workers stack.
- **C (JHM base, rookiecards as reference only)** — same problem as B, plus losing institutional knowledge of every email template, every status transition, every consent-tracking detail. Highest churn, highest regression risk.
- **D (Hybrid free-form)** — collapses into A in practice once you list the constraints. Option A *is* the disciplined hybrid: take only the design from JHM, keep all of the runtime from rookiecards.

### What gets sourced from where

**From `junior-hero-moments` (design only):**
- Visual design tokens — colours, gradient definitions, font choices (Anton + Inter + JetBrains Mono), shadow/glow recipes, the `text-gradient-gold` and `bg-hero` utility classes — translated into the **existing Tailwind 3 config** (do not bring Tailwind 4 over in this pass).
- shadcn/ui component library — **install fresh** via the shadcn CLI into `client/src/components/ui/` rather than copy/pasting JHM's files (avoids Tailwind 4 / React 19 coupling and pulls only what's needed).
- 5 hero photographs from `src/assets/` → `client/public/` or `client/src/assets/`.
- Page-level visual layout reference — read JHM's `routes/index.tsx` and re-skin `pages/ClubLanding.tsx` accordingly. The "FOUNDING 75 of N spots left" badge becomes a marketing decision (see §4 risks).
- `framer-motion` motion patterns from JHM hero — but rookiecards already has `framer-motion@^12.38` in `client/`, so reuse the existing install.

**Kept from `rookiecards` (everything else):**
- Entire `server/` workspace (Express, SQLite, Stripe, Resend, composite pipeline, cron jobs, all 10 routes).
- All `client/src/pages/*` (re-skinned, not rewritten — keep state machines, API calls, validation logic).
- `client/src/lib/api.ts`, `lib/compress.ts`, `lib/guides.ts` — unchanged.
- MDX guides system (vite config, content/, feeds plugin).
- Brand-asset, marketing, design-system, docs, pipeline folders — unchanged.
- Deployment topology (Cloudflare Pages + Railway).

### Files / folders to copy

Copy **from `junior-hero-moments` → `rookiecards/client`:**
| Source | Destination | Notes |
|---|---|---|
| `src/assets/*.jpg` | `client/src/assets/` (or `public/`) | New hero/box/three-tiers photography |
| Selected fragments of `src/components/site/` | inspirational — rewrite in rookiecards' style | Don't copy verbatim — JHM Header uses TanStack `Link`, not `react-router-dom` |
| Color/font/shadow declarations from `src/styles.css` | merge into `client/tailwind.config.js` + `client/src/index.css` | Translate Tailwind 4 `@theme` blocks → Tailwind 3 `extend` |
| `src/lib/site.ts` constants (if mechanic is kept) | new `client/src/lib/launchConstants.ts` | See §risks — confirm with stakeholder before keeping FOUNDING 75 |

**Do NOT copy:**
- `routes/` (TanStack file routes — incompatible with rookiecards router).
- `routeTree.gen.ts`, `router.tsx`, `server.ts`, `start.ts`, `wrangler.jsonc` (Workers SSR — wrong deployment target).
- `components/ui/*` (re-add via shadcn CLI to match installed Tailwind/React versions).
- `package.json` (would clobber the monorepo definition).

### Dependencies — what to add / remove / upgrade

In `client/package.json`:
- **Add (prod):** `class-variance-authority`, `clsx`, `tailwind-merge`, plus only those `@radix-ui/react-*` primitives the new design actually consumes (probably ~10 of the 28 JHM ships).
- **Add (dev):** none from JHM (the Lovable plugin, Cloudflare plugin, ESLint stack are not needed).
- **Fix:** pin `lucide-react` to a sane modern version (e.g. `^0.460.0`) — investigate why `^1.7.0` is currently in `package.json`; it is almost certainly an incorrect pin and may currently resolve to a long-deprecated build.
- **Defer:** Tailwind 4 upgrade, React 19 upgrade — both major and should be separate work-streams, not bundled into the design merge.
- **Defer / decide:** `react-hook-form` + `zod` — only add if rewriting `OrderFlow.tsx`'s form layer is in scope.

In `server/package.json`: **no changes.**

### Conflicts to resolve
1. **Tailwind 3 vs 4 design tokens** — JHM uses Tailwind 4's `@theme { --color-primary: oklch(...) }` syntax. These must be translated to Tailwind 3's `theme.extend.colors.primary = '#…'` form. Plan an hour for token translation.
2. **shadcn/ui assumes Tailwind 4 + CSS vars in JHM** — when re-installing via CLI into rookiecards, configure for Tailwind 3 + the existing `index.css`. Use `style: "default"` (not `"new-york"`) unless we deliberately want to keep the new-york look — both work on Tailwind 3 but visual details differ.
3. **Routing** — purely a JHM-side concern; ignore TanStack assets entirely.
4. **`lucide-react` API** — once pinned, sweep `client/src/` for icon imports and verify each icon name still exists at the new version.
5. **Font loading** — JHM loads Anton + Inter from Google Fonts in `__root.tsx`. Add equivalent `<link>` tags to `client/index.html`.
6. **FOUNDING 75 mechanic vs Charter Partner mechanic** — these are two different launch narratives. Marketing/product decision required; see §risks.

### Order of operations
1. **Branch off `rookiecards/main`** as `merge/jhm-design`. Do not start from scratch.
2. **Install design-token foundations:** update `client/tailwind.config.js` with JHM's colours, fonts, shadows, keyframes; add Google Fonts `<link>` tags to `client/index.html`.
3. **Set up shadcn/ui:** run `npx shadcn@latest init` in `client/`; configure paths to match `src/components/ui/`, `src/lib/utils.ts`. Install only `button`, `input`, `label`, `card`, `dialog`, `dropdown-menu`, `toast`/`sonner`, `accordion`, `progress`, `select`, `textarea`, `tooltip` initially.
4. **Build a new `Header` + `Footer`** in `client/src/components/` mirroring JHM's visual language but using `react-router-dom`'s `Link`. Swap them into the existing pages.
5. **Re-skin `ClubLanding.tsx`** first — single biggest visual win, sets the language for everything else. Keep all existing logic; replace only JSX/className.
6. **Re-skin `OrderFlow.tsx`** — *do not rewrite*. Replace step shells, form fields, and progress UI with shadcn equivalents; keep state, API calls, validation, redirect logic untouched.
7. **Re-skin `ClubParentLanding`, `ParentLanding`, `Confirmation`, `SharePage`** in that order.
8. **Re-skin `InterestForm`, `ContactPage`** — these are simpler forms; consider adding `react-hook-form` + `zod` if it's a clear win, but do not require it.
9. **Leave `AdminDashboard` and the guides system for last** — admin can stay rough; guides have MDX coupling that's easy to break.
10. **Decide on the launch mechanic** (FOUNDING 75 vs Charter Partner — see §risks) before merging the home page.
11. **Visual QA in a real browser** at every step. Run `npm run dev` from `rookiecards/` root (boots client on :5173 + server on :3001). Stripe test mode with `4242 4242 4242 4242`. Resend in dev — leave `RESEND_API_KEY` blank to no-op.
12. **Smoke-test the full live flow** (parent lands → orders → checks out → webhook fires → confirmation email → share link) on a preview deploy *before* merging to `main`.

### Estimated complexity — **medium**
About **3–5 days of focused work**, broken roughly:
- 0.5 day: design-token translation + shadcn init + Header/Footer.
- 1 day: re-skin home + parent landing.
- 1–1.5 days: re-skin OrderFlow (the riskiest single file; visual changes only, no logic touch).
- 0.5 day: re-skin Confirmation, Share, InterestForm, Contact.
- 0.5 day: cross-page QA, mobile QA, regression sweep.
- 0.5 day: deploy preview + end-to-end smoke test.

This estimate assumes the launch-mechanic decision is made up front and no Tailwind 4 / React 19 / TanStack Router migration is bundled into this PR.

### Risks — what could break, what's untested

| Risk | Severity | Mitigation |
|---|---|---|
| **`OrderFlow.tsx` regression** — 1 042 LOC, no tests, payment-critical. | 🔴 High | Re-skin in a side-by-side branch, run through Stripe test mode end-to-end before merge. Keep all state/handlers identical; touch only JSX. |
| **Stripe webhook signature verification** — relies on raw body parsing in `server/src/index.ts`. | 🔴 High | Out of scope for the design merge — but worth verifying after deploy that webhooks still arrive. |
| **MDX guides build** — `vite.config.ts` has a custom `rawMdxPlugin` plus a feeds plugin. Tailwind changes could break `prose` styling. | 🟡 Medium | Don't remove `@tailwindcss/typography`. Render `/guides` after each Tailwind config edit. |
| **`lucide-react` icon renames** — current `^1.7.0` pin is suspicious; upgrading may rename icons used across 11 pages. | 🟡 Medium | Pin first, then grep for icon imports and verify each compiles. |
| **Launch-mechanic mismatch** (FOUNDING 75 vs Charter Partner) — these are *different stories*. Mixing them on the home page would confuse parents and clubs. | 🟡 Medium | Make this a product decision before re-skinning the home page. Recommend keeping the existing **Charter Partner / 10% kickback** narrative — it's the one paid customers and registered clubs already know — and treating "FOUNDING 75" as a visual idea that needs a real mechanic if it's kept. |
| **No test coverage in either repo** — no Jest/Vitest/Playwright in either `package.json`. Every change is hand-verified. | 🟡 Medium | Mandatory manual checklist for each re-skinned page: load, validate, submit, redirect, email arrival. |
| **React 19 / Tailwind 4 / TanStack Start migration creep** — easy to drift into "let's also upgrade…". | 🟡 Medium | Hold the line: design merge in one PR; framework upgrades in separate PRs. |
| **Email rendering** — Resend templates use inline-styled HTML strings. Brand colour changes (`#0B1D3A` → new primary) must flow into `email.ts` too, or the new branding stops at the website. | 🟢 Low | Sweep `server/src/lib/email.ts` once design tokens are locked. |
| **Cloudflare Workers vs Pages** — JHM is built for Workers (SSR). If anyone confuses the two, deployment breaks. | 🟢 Low | Don't copy `wrangler.jsonc`. Keep `client/cloudflare-pages.json` authoritative. |

---

## 5. Open questions for the reviewer

1. **Launch mechanic** — do you want to keep "FOUNDING 75" (and if so, what does it actually *do* — refund, lottery entry, discount?), or stay with the existing "Charter Partner Clubs / 10% kickback / first 50 clubs" narrative? The JHM design references the former; the live app implements the latter.
2. **Founding-75 incentives the brief mentions** — "refund-for-UGC, $500 prize comp." Neither exists in either repo's code. Are these meant to be built as part of this merge, or just referenced in copy?
3. **`lucide-react@^1.7.0`** — confirm this is intentional? It is a very unusual major version (most projects sit on `^0.x`); if it resolves to a long-stale build, it's worth fixing as part of the same PR.
4. **Tailwind 4 / React 19 / TanStack Start upgrades** — confirm these are *out of scope* for the design merge? My plan assumes yes.
5. **shadcn/ui style** — JHM uses `"new-york"` style. Do you want that exact look, or `"default"` (which is more rounded/softer)?
6. **Guides system** — keep as-is? Or re-skin the guide article templates too as part of this pass?
7. **Admin dashboard** — re-skin in this pass, or leave on the old visual language until later?

---

## 6. Blockers & install/build report

**No blockers encountered.** Both repos:
- Cloned cleanly.
- Installed cleanly on Windows (PowerShell, no admin shell needed).
- Built cleanly to production output.

Surprises worth noting:
- `rookiecards/client` ships **`lucide-react@^1.7.0`** in `package.json` — version `1.7.0` of `lucide-react` does not exist on npm (the project is on `0.x` releases as of writing). npm is silently resolving this to something — almost certainly an unintended pin. **Recommend fixing as part of (or before) the design-merge PR.**
- `rookiecards/server` references a `triggerPipeline` function (`server/src/lib/pipeline.ts`) that calls a legacy n8n instance. New work goes through `/api/composite/*` (Replicate Flux Kontext). The legacy path is kept for in-flight orders and silently no-ops when `N8N_WEBHOOK_BASE_URL` is unset.
- `rookiecards` server build warns of a 538 kB client bundle and `rookiecards/server` ships an unused `nodemailer` dependency alongside Resend (`@types/nodemailer` is on the prod dep list — minor cleanup, not a blocker).
