# Sketrix

Offline-first canvas built on Excalidraw for dumping messy notes (text, screenshots, sketches), plus AI that turns them into clean structured notes. Web + mobile, synced. Personal daily-use tool first.

Name status: Sketrix is chosen. Availability (domains, npm, app stores, handles, trademark) is still being checked, so do not hard-code the name in places that are costly to change (package scopes, bundle IDs) until I confirm.

## Vocabulary
- **Sketch** = the canvas where I dump.
- **Matrix** = the AI-organized view.
- **Tidy** = the action that turns a Sketch into a Matrix.
- **Boards** = saved canvases.
- **Sketrix AI** = the user-facing name of the AI feature. The product itself is just "Sketrix".

## Current phase
**Phase 0: AI eval only.** No app code until the eval says which model is good enough.
0. AI eval (script over my real `.excalidraw` files)
1. Web app: embedded canvas, local save, Matrix view
2. Accounts, cloud sync, image upload
3. Mobile (Expo)
4. Live sharing sessions

Do not build ahead of the current phase.

## Decisions (do not change without asking me)
- Embed `@excalidraw/excalidraw` (MIT). Never fork or rebuild the canvas. Client-only, no SSR: web is a Vite SPA, not Next.js.
- Monorepo: pnpm workspaces. `apps/web`, `apps/api`, `apps/mobile` (later), `packages/shared`. No Turborepo until builds are slow.
- Backend: Node + Express + TypeScript, zod at every boundary, Postgres + Prisma. A board's scene is one JSONB blob, not a row per element.
- Images: never store base64 images inside the scene. Store them separately (S3) by Excalidraw file id, with a local blob cache for offline.
- Mobile (Phase 3): Expo + `react-native-webview` loading the **bundled** web canvas (so it works offline), talking to the native shell via `postMessage`.
- Sync (Phase 2): local-first, no CRDT and no sync engine in v1. Merge per element: higher `version` wins, tie goes to lower `versionNonce`. Keep `isDeleted` tombstones. The merge function lives in `packages/shared` and is used by both client and server. It must have unit tests.
- `onChange` fires constantly. Debounce saves and save on pointer-up.
- No end-to-end encryption on personal boards (server-side AI needs to read text).

## AI rules
- v1 reads typed text elements only (no handwriting, no vision).
- Cluster text elements by position, sort top-to-bottom, send one cluster per call.
- The raw canvas is the source of truth. AI never edits it. Output goes to the Matrix view, each item linked to its source element ids.
- Prompt contract: fix grammar and structure only. NEVER add facts, names, roles or details I did not write. Anything ambiguous goes into an `uncertain` list instead of being guessed. (Past failure: "engineer" became "Quality Engineer".)
- Triggers: idle debounce or the Tidy button, never per keystroke. Cache results by cluster content hash. Per-user daily cap.
- All AI calls go through one provider-agnostic `organize(cluster)` function returning JSON validated by zod.
- Candidate models, to be chosen by eval on real notes: Groq `llama-3.1-8b-instant`, Groq `llama-3.3-70b-versatile`, Groq `gpt-oss` models, Gemini 3.1 Flash-Lite (paid). Pick the cheapest one that never invents facts.
- Privacy: enable Zero Data Retention on Groq. Do not use any free tier that trains on inputs for work notes. Never commit real meeting notes to git; sample data lives in a gitignored `samples/` folder.

## Conventions
- TypeScript strict. pnpm only. Small commits. Ask before adding a dependency.
- Keep it simple. If a change adds infrastructure, say so and wait for my go-ahead.