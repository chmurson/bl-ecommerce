# Agent Handbook

## TL;DR
- Yarn 4 (Berry) with Plug'n'Play is the package manager; never add `node_modules` or run `npm install`.
- Biome handles all linting and formatting. `yarn lint` will rewrite files in place to satisfy the style guide.
- The user-facing surface lives in `pckgs/site` (Next.js App Router). The headless CMS for content lives in `pckgs/cms` (Strapi); expect to deploy it separately from the site.
- The Next.js app is stateless: catalog data is fetched from the CMS, checkout is powered by Stripe, and there is no application database.
- There are no automated tests yet. Add new tests under `pckgs/site/src/__tests__/` and mirror the module path (`format-price.ts` → `format-price.test.ts`).
- Commit messages follow Conventional Commits (`feat:`, `fix:`, etc.). Run `yarn commitlint --from HEAD~1` locally if you need to double-check formatting.

## Repository Layout
| Path | Description |
| --- | --- |
| `.` | Shared configuration (`biome.json`, `.editorconfig`, Turbopack config) and the Yarn Berry workspace root. |
| `pckgs/site` | Next.js 15 App Router project. Anything shipped to production should live here under `src/` with feature routes segmented inside `src/app/…`. |
| `pckgs/site/src/app` | App Router entry point. Use nested segments (`(cart)/summary`) to group feature-specific UI. Shared hooks and utilities belong under `src/lib/`, reusable UI under `src/components/`. |
| `pckgs/cms` | Strapi-based CMS used for product data and rich content. Primarily for local development but will eventually be hosted independently from the site. |
| `public/` | Static assets shared by the site (favicons, OG images, etc.). Reference them relatively from Next.js routes. |

## Tooling
- **Package manager:** Yarn 4.9.2 in Plug'n'Play mode. Use `yarn install` (or `yarn install --immutable` in CI) to sync dependencies. Do not check in `node_modules/` directories—Strapi may generate one locally, but PnP is still the source of truth.
- **Runtime:** The site targets the latest Next.js and React (app router). Align Node versions via `.nvmrc` if one is added; otherwise follow the version used in CI.
- **Formatting & linting:** Biome is the single source of truth. `yarn lint` runs `biome check … --write` inside the `site` workspace. Use `yarn workspace site lint-fix` for an aggressive rewrite or `yarn workspace site lint --no-write` once that script exists. Ignore `eslint` unless you are updating legacy rules.
- **Editor settings:** `.editorconfig` enforces UTF-8, LF, two spaces, trailing newline. Biome uses a 160 char line width and double quotes by default.

## Scripts & Workflows
- `yarn dev` – Launches the Next.js site (http://localhost:3000).
- `yarn build` / `yarn start` – Production bundle + start via `next`.
- `yarn lint` – Runs Biome with write mode against the site workspace.
- `yarn commitlint` – Lints commit history using Conventional Commit rules (pass `--from` / `--to` as needed).
- `yarn cms:dev` – Starts the Strapi CMS locally (defaults to http://localhost:1337).
- `yarn cms:build` / `yarn cms:start` – Build and serve the CMS for production verification.
- `yarn cms:install` – Installs CMS dependencies (required after pulling upstream Strapi updates).

When working on features:
1. Run `yarn install` once per pull to sync PnP.
2. Start the app with `yarn dev`. Use the CMS in parallel if the feature touches catalog content (`yarn cms:dev`).
3. Keep CMS schema changes coordinated. Export Strapi configs to source control (see `pckgs/cms/config` and `pckgs/cms/src`).
4. Use Stripe test keys for checkout flows. Store secrets in `.env.local` for the site and `.env` for the CMS; do not commit them. Document any new required variables in the README and in this file.

## Testing & QA
- There is no automated test suite yet. If you add one, colocate tests inside `pckgs/site/src/__tests__/` and follow the `{module}.test.ts` naming pattern.
- Prefer Vitest (works well with Next.js 15) or Jest. Expose the runner via `yarn test` at the root (`yarn workspace site test`) to match Yarn Plug'n'Play expectations.
- Until tests exist, rely on manual QA: run `yarn build && yarn start` before handing off and smoke the main flows (catalog browse, cart, checkout → Stripe redirection).

## CMS Notes
- The CMS exports product data consumed by the site. Add content types and seed data under `pckgs/cms/src` so they are versioned.
- The CMS may leverage a SQLite dev database under `pckgs/cms/.tmp` or similar. Keep those directories out of git.
- For hosted environments, expect to deploy the CMS separately (e.g., Render, Railway). Provision environment variables and storage (S3, Cloudinary) in that context; the site will consume CMS APIs over HTTPS.

## Stripe & External Services
- Checkout is handled client-side via Stripe. Keep Stripe keys in environment variables (`NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`, `STRIPE_SECRET_KEY`). Never commit credentials.
- If you add serverless handlers, put them under `pckgs/site/src/app/api/…`. Use the CMS as the source of truth for catalog data rather than persisting inside the Next.js app.

## Pull Requests & Releases
- Keep commits in Conventional Commit format (`feat: add product card`). Reference issues in the body (`Refs #123`).
- PRs must describe the change, include screenshots for UI updates, note manual verification steps, and call out risks or rollout considerations (CMS migration, Stripe config, etc.).
- Run `yarn lint` and, once available, `yarn test` before requesting review. Attach logs or screenshots for significant flows (Stripe checkout, CMS schema changes).
- Coordinate CMS releases with site deploys. Version schema changes and content migrations so they can be applied predictably in staging and production.

## Getting Help
- Document tribal knowledge in this file or the README as it surfaces.
- When unsure about CMS schema, coordinate with the content team before refactors.
- If you encounter PnP resolution issues, run `yarn install --check-cache` and commit the updated `.yarn/` metadata if it changes.

Keep this handbook up to date as the workflows evolve. Aim for clarity and reproducibility so future agents can ship confidently.
