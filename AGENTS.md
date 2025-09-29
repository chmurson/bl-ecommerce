# Repository Guidelines

## Project Structure & Module Organization
Keep all production code under `src/`, using the App Router in `src/app/` to compose routes and layouts. Group feature-specific UI in nested route segments (`src/app/(cart)/summary`) and keep shared utilities in `src/lib/` while reusable components live in `src/components/`. Store automated tests alongside features in `src/__tests__/` with filenames mirroring the module under test (`src/lib/format-price.ts` -> `src/__tests__/format-price.test.ts`). Static assets (logos, favicons, Open Graph images) belong in `public/` and should be referenced relatively. Configuration files (.editorconfig, eslint config, Tailwind, etc.) stay at the repository root so Yarn Plug'n'Play (`.pnp.cjs`) continues to resolve modules deterministically.

## Build, Test, and Development Commands
Install dependencies with `yarn install`; Yarn 4 runs in Plug'n'Play mode, so avoid `npm install`. Define runnable scripts in `package.json` and invoke them via `yarn <script>`. Typical examples:
- `yarn dev` — start the local development server at http://localhost:3000.
- `yarn build` — produce a production-ready bundle with Turbopack.
- `yarn lint` — run the flat-config ESLint suite (Core Web Vitals rules).
- `yarn test` — execute the automated test suite once it is added.
If a script touches Node binaries directly, prefer `yarn node <file>` or `yarn exec <tool>` so dependencies load through PnP without a `node_modules` folder.

## Coding Style & Naming Conventions
Respect the root `.editorconfig`: UTF-8, LF line endings, two-space indentation, and a trailing newline. Keep TypeScript/JavaScript files in ES module syntax and name React components in PascalCase (`ProductList`). Utility functions use camelCase (`formatPrice`). Commit generated files or build artefacts only when explicitly required. When formatting help is needed, add a `format` script that runs Prettier (`yarn format`).

## Testing Guidelines
Adopt a single test runner (Vitest or Jest recommended) and expose it via `yarn test`. Co-locate unit tests within `src/__tests__/` and name them `*.test.ts`. Integration scenarios should document expected fixtures under `src/__tests__/__fixtures__/`. Maintain fast feedback by running `yarn test --watch` before pushing. Target ≥80% line coverage and document any intentional gaps in the pull request description.

## Commit & Pull Request Guidelines
Follow the existing short, imperative commit style (`init`); keep summaries under 50 characters and add descriptive body text when necessary. Reference issue IDs in the body (`Refs #123`). Pull requests must describe the change, include screenshots or terminal output for UI/API changes, list test coverage, and call out risks or rollout steps. Request review from at least one teammate before merging and wait for CI to pass.
