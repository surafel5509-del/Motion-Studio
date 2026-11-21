# MASTER BUILD PROMPT — MOTION STUDIO

You are the principal engineer and technical product architect responsible for turning this repository into a professional, production-grade motion graphics editor. The goal is a substantial, maintainable application inspired by the workflow depth of Adobe After Effects, not a clone of its proprietary implementation. Read README.md and docs/architecture/OVERVIEW.md completely before making changes.

## Mission

Build Motion Studio as a real, mobile-first and desktop-capable motion graphics platform. It must support actual projects, compositions, layers, frame-accurate timeline editing, keyframes, non-destructive effects, media import, GPU preview, reliable rendering/export, persistence, and a secure MCP server for AI agents. It should be architected for future professional features such as masks, precompositions, graph editor, 3D layers, collaboration, plugin SDK, and render workers.

Do not stop at a polished static UI. Do not create fake buttons, fake progress bars, fake exports, demo-only state, or placeholder controls presented as finished features. Every implemented feature must have a complete user workflow, tests, and documentation.

## Mandatory first actions

1. Inspect all repository files, git history, package manifests, workflows, and existing code. Do not assume this is an empty repository.
2. Report current state: what works, what is missing, risks, build status, and existing architecture.
3. Create a prioritized plan with file-level changes, dependencies, acceptance criteria, and risks.
4. Create/update architecture decision records for consequential technology choices.
5. Implement Phase 0 foundation and run checks before adding broad UI features.
6. Never delete or replace user work without explaining why and preserving a recovery path.

## Required architecture

Use a TypeScript monorepo with pnpm workspaces and Turborepo. Recommended apps: apps/web (Next.js/React editor), apps/mcp-server (MCP service), apps/render-worker (render/encode workers), and optional apps/api. Shared packages must include project-schema, editor-core, timeline, animation, scene-graph, renderer, effects, media, audio, export, plugin-sdk, mcp-contracts, ui, config and telemetry.

Keep the React shell separate from timeline math, project state, scene evaluation, renderer, and persistence. The versioned project document is the source of truth. All editing must go through typed application commands. Both UI and MCP call the same command services. Use strict TypeScript, Zod validation at trust boundaries, stable IDs, rational frame rates, integer frame positions, and explicit schema migrations.

## Product requirements

### Projects and compositions
Create, open, save, rename, duplicate, delete, import and export projects. Support compositions with width, height, rational FPS, duration in frames, color settings, and background. Use IndexedDB for local projects and media. Add crash recovery and autosave before cloud sync. Project/media data must survive browser reloads.

### Timeline and layers
Implement multi-track timeline with layer rows, clips, frame ruler, playhead, scrubbing, zoom, snapping, markers, in/out points, selection, reorder, trim and split. Support text, shape, image, video, audio, null and precomposition layer types incrementally. Add visibility, lock, solo, parenting and blend modes when their behavior is real. Use integer frames internally. Touch gestures must be designed separately for mobile.

### Animation
Support transform properties (anchor, position, scale, rotation, opacity), keyframes, hold/linear/Bezier interpolation, easing, and an actual graph editor as a later milestone. Build pure deterministic interpolation functions with thorough tests. Undo/redo must include keyframe operations.

### Renderer
Build a renderer-neutral scene graph and renderer interface. Start with a functional 2D renderer and GPU preview (WebGL2 baseline, WebGPU enhancement). Add effects through a typed effect graph. Preview and final render must share evaluation semantics. Cache by project revision/frame/assets/effect parameters. Handle device loss, cancellation, memory limits and fallback. Never pretend that a CSS transform preview is equivalent to final compositing if it is not.

### Media, audio and export
Import media using signature/type validation and isolated decoding. Generate thumbnails and metadata; support relinking and proxies. Use Web Audio for real audio playback and waveform data. Export must render actual frames and encode valid media, with progress, cancellation, errors and resumability. Clearly disclose browser codec limits; use server-side FFmpeg workers for formats that cannot be reliably encoded in-browser. Never claim MP4 or audio export unless verified by tests.

### Persistence and cloud
Local-first IndexedDB storage must work without a paid service. Add cloud sync only with explicit auth, PostgreSQL metadata, object storage, versioning, optimistic concurrency, conflict handling, signed uploads and tenant isolation. Do not store media blobs in localStorage.

### MCP and agent workflows
Implement a separate MCP server with the official MCP TypeScript SDK. Support stdio for local integrations and authenticated Streamable HTTP for hosted integrations. Expose read-only resources/tools separately from editing/rendering tools. Use shared Zod contracts. MCP must invoke the same application services as the UI.

At minimum plan resources for project, composition, timeline, assets, render jobs, schema and effects catalog. Plan tools for list/get/validate projects; inspect compositions/layers/assets; create projects/compositions; add/update/remove/reorder layers; set properties; add/move keyframes; trim/split clips; add/update effects; undo/redo; relink assets; preview frames; validate/start/cancel render; retrieve authorized artifacts. Add prompts for title animation, explainer video, social format adaptation and project review.

Security is mandatory: authenticate remote requests; enforce tenant/project scopes; validate ownership; use idempotency keys; rate-limit; cap payloads; structured errors; audit actions; short-lived signed URLs; confirmation gates for deletion, destructive edits, publishing, sharing and expensive renders. No arbitrary shell, filesystem, URL fetch, or unrestricted code execution. Expressions and third-party plugins must be sandboxed.

### UX
Desktop should offer a professional workspace with media/effects browser, composition viewer, inspector and timeline. Mobile must use focused panels/bottom sheets, touch-sized controls, pinch zoom, pan and touch-safe timeline gestures. Do not squeeze desktop UI into a phone. Every interaction needs keyboard/touch accessibility and clear states.

### ChatGPT compatibility
Treat MCP as the primary AI protocol. Add an optional OpenAPI adapter for compatible app/connector ecosystems. Do not target the retired legacy ChatGPT Plugins model as if it were a currently supported universal installation path. Keep the protocol boundary provider-neutral.

## Delivery sequence

Phase 0: monorepo, CI, lint, typecheck, tests, design tokens, project schema, config, docs, ADRs.
Phase 1: complete editing vertical slice: projects, composition, layers, timeline, text/shape/image, transforms, keyframes, undo/redo, IndexedDB.
Phase 2: real scene graph, GPU preview, effects, media decode, cache and deterministic render tests.
Phase 3: real audio, proxies, render worker, export queue and tested output formats.
Phase 4: graph editor, masks, parenting, precomps, blend modes, text animation, color management, motion blur, 3D foundations.
Phase 5: auth, cloud persistence, sync, collaboration and version history.
Phase 6: MCP resources/tools/prompts, OAuth, scopes, approval gates, audit, examples and evaluations.
Phase 7: plugin SDK, render farm, advanced effects, templates, quotas, performance and security hardening.

Ship each phase as a usable, tested vertical slice. If scope exceeds the available time, finish a smaller end-to-end feature instead of scaffolding dozens of incomplete panels.

## Engineering standards

- Strict TypeScript; avoid any and unsafe casts.
- Use small modules, clear boundaries, stable interfaces and dependency injection where useful.
- No secrets, tokens, private media, or real credentials in git.
- Validate all external data and imported project files.
- Use migrations for persisted schema changes.
- Unit-test pure logic; integration-test application services; Playwright-test core workflows.
- Test keyboard, touch, narrow viewport, slow device and error states.
- Run pnpm lint, pnpm typecheck, pnpm test, pnpm test:e2e (where configured), and pnpm build before claiming success.
- Fix failures; do not suppress tests or lower type safety to make CI green.
- Add accessibility labels and keyboard paths.
- Keep dependencies maintained and pinned; document licensing.
- Document limitations honestly. A feature is done only when its workflow, tests and docs are present.

## Definition of done for each feature

A feature is complete only when: schema and command are defined; permissions are checked; UI is wired; state persists; undo/redo behavior is specified; error/loading/empty states exist; unit/integration/E2E tests cover it; accessibility is addressed; docs are updated; and a clean build passes.

## Final instruction

Start by auditing this repository and reporting the plan. Then implement Phase 0. Continue with Phase 1 as the first complete editing slice. Do not claim that Motion Studio is a full After Effects replacement until the required capabilities genuinely exist and have been verified.
