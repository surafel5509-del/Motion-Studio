# Motion Studio — Professional Motion Graphics Platform

A mobile-first, desktop-capable motion graphics and compositing platform with an After Effects-inspired workflow, deterministic rendering, extensible effects, cloud projects, and MCP automation.

- Repository: https://github.com/surafel5509-del/Motion-Studio
- Product owner: Surafel Seifu
- Status: architecture blueprint and project initialization; not yet a production-complete editor.

## Vision and principles

Motion Studio must be a real editing application, not a visual mockup. It needs a canonical project model, editable frame-accurate timeline, non-destructive effects, media management, deterministic preview/rendering, reliable export, and an AI/MCP interface that uses the same validated application commands as the UI.

The workflow may be inspired by professional compositing tools such as Adobe After Effects, but all implementation, branding, UI, code, icons, and assets must be original. Never copy proprietary source code or assets.

1. Real editing: every visible control changes actual state and is tested.
2. Non-destructive workflow: source media is never overwritten by transforms/effects.
3. Deterministic projects: same project/assets/fonts/settings yield reproducible output within documented platform tolerances.
4. Mobile-first, desktop-powerful: touch-first phone/tablet workflows plus dense desktop panels and shortcuts.
5. Local-first with optional cloud sync and collaboration.
6. AI is an interface, not the engine: MCP and UI call the same command layer.
7. Accessibility, privacy, security, observability, recovery, and clear errors are product requirements.
8. Deliver complete vertical slices, not empty panels or fake controls.

## Product scope

### Core editor
- Projects and compositions: dimensions, pixel aspect, rational FPS, duration in frames, color settings, background and preview quality.
- Multi-track timeline: video, audio, text, vector shape, camera, light, null, adjustment and precomposition layers.
- Layer order, parenting, grouping, precomps, track mattes, blend modes, visibility, solo, lock, shy, guides and labels.
- Frame-accurate playhead, snapping, markers, work area, in/out points, trim, split, ripple, slip, slide, time stretch and nested sequences.
- Transform properties: anchor point, 2D/3D position, scale, rotation, orientation, opacity, skew and motion paths.
- Keyframes: hold/linear/Bezier, temporal and spatial interpolation, easing, velocity handles, roving keyframes and graph editor.
- Text engine: font fallback, point/paragraph text, alignment, tracking, leading, kerning, fill, stroke, text-on-path and per-character animation.
- Vector shapes: Bezier paths, pen, masks, trim paths, repeaters, gradients, strokes, boolean operations and shape animators.
- Effect stack: blur, glow, shadow, curves, levels, hue/saturation, chroma key, displacement, distortion, motion blur, grain, sharpen and custom effect SDK.
- Audio import/playback, waveform peaks, gain, pan, fades, meters and sync.
- Media import: images, SVG, video, audio, fonts, image sequences, proxies, relinking, metadata, thumbnails and cache.
- Transactional undo/redo, autosave, crash recovery and version history.
- Preview quality controls, frame cache, draft/final rendering, region-of-interest and diagnostics.
- Export queue, presets, ranges, alpha, image sequences, audio, and supported MP4/WebM/ProRes formats when encoder/runtime supports them.

### Advanced roadmap
3D layers/cameras/lights; masks and mattes; motion blur; frame blending; time remapping; color management; sandboxed expressions; plugin/effect SDK; templates and procedural generators; proxy workflow; render workers/farm; collaboration, comments, presence and permissions; AI storyboard-to-timeline and prompt-to-animation.

Do not claim After Effects feature parity. Do not advertise advanced 3D, optical flow, rotoscoping, color science, or codec support until implemented and tested. Gate incomplete features behind capability flags.

## Recommended stack

Use a strict TypeScript monorepo. Select stable compatible versions at implementation time and pin them.

| Concern | Recommended baseline |
|---|---|
| Monorepo | pnpm workspaces + Turborepo |
| Web app | Next.js, React, TypeScript |
| State and validation | Zustand or Redux Toolkit; Zod |
| Timeline | Custom typed, frame-based model |
| 2D renderer | WebGL2 initially; WebGPU progressive enhancement; PixiJS or custom scene graph |
| Video | WebCodecs where supported, fallback adapter |
| Audio | Web Audio API |
| Export | WebCodecs/muxer adapter plus server-side FFmpeg worker |
| Compute | Web Workers |
| API | Next.js route handlers or Fastify service |
| Database | PostgreSQL with Prisma or Drizzle |
| Media storage | S3-compatible object storage |
| Jobs | Redis + BullMQ or managed queue |
| Realtime | WebSocket/WebRTC provider adapter |
| MCP | Official MCP TypeScript SDK, separate service |
| Tests | Vitest, Playwright, Testing Library |
| Observability | OpenTelemetry and structured logs |
| Hosting | Vercel for web shell; container compute for render workers |

### Target monorepo

    apps/web                 Next.js dashboard and editor
    apps/mcp-server          MCP resources, tools, prompts, auth
    apps/render-worker       Headless render and encoding workers
    apps/api                 Optional dedicated API service
    packages/project-schema Versioned project format and Zod schemas
    packages/editor-core    Commands, history, selection, snapping
    packages/timeline       Timeline math and interactions
    packages/animation      Keyframes, interpolation, expressions
    packages/scene-graph    Nodes, transforms, parenting, compositing
    packages/renderer       Renderer interfaces and GPU backends
    packages/effects        Built-in effects and effect SDK
    packages/media          Import, metadata, proxies, thumbnails
    packages/audio          Audio graph and waveform utilities
    packages/export         Render plans, encoders, presets
    packages/plugin-sdk     Third-party extension contracts
    packages/mcp-contracts  MCP schemas and permissions
    packages/ui             Shared components and design tokens
    packages/config         Shared TS/lint/test configuration
    packages/telemetry      Logging and tracing
    docs/architecture       Architecture and diagrams
    docs/decisions          Architecture decision records
    docs/mcp                MCP reference and examples
    docs/security           Threat model and security controls
    docs/testing            Test strategy and quality gates
    examples/projects       Sample project documents
    examples/plugins        Example effect plugins
    examples/mcp-clients    MCP client examples

## Architecture contracts

### Canonical project document
The versioned project document, not React state, is the source of truth. Model Project, Composition, Layer, Property, Keyframe, Asset, Effect, Marker, ExportPreset and RenderJob. Use stable IDs, explicit units, schemaVersion and migrations. Store timeline positions as integer frames and FPS as a rational numerator/denominator; convert to seconds only at I/O boundaries. Every imported project and command must be runtime-validated.

### Command/application layer
All edits pass through typed commands such as AddLayer, RemoveLayer, SetProperty, AddKeyframe, MoveKeyframe, TrimLayer, SplitLayer, ReorderLayer, AddEffect, SetCompositionSettings and RelinkAsset. Each command validates preconditions/permissions, applies atomically, returns inverse data for undo, emits change events for autosave/collaboration, is serializable for MCP/audit, and has deterministic tests. UI and MCP must never mutate project JSON directly.

### Rendering
Create a renderer-neutral scene graph and backend interface. Start with 2D GPU compositing, then add WebGPU and 3D incrementally. Evaluation at a frame must resolve layers, parenting, transforms, keyframes, effects, blend modes, color and alpha. Cache keys include project revision, frame, asset hashes and effect parameters. Rendering supports cancellation and progress. Preview and final export share scene evaluation and effect semantics; quality differences are explicit and tested.

### Persistence and collaboration
Use IndexedDB for local drafts and media; do not rely on localStorage for large projects. Cloud metadata belongs in PostgreSQL and media in object storage. Implement autosave checkpoints, recoverable operation logs, optimistic concurrency with revisions/ETags, documented conflict resolution, signed resumable uploads, checksums, quotas, asset relinking, and project bundle import/export. Never silently overwrite concurrent edits.

### UX
Desktop: top bar for project/save/undo/redo/preview/export; left media/effects/templates browser; center composition viewer; right inspector; bottom timeline/layer/property/graph editor. Panels are resizable, collapsible and keyboard accessible.

Mobile: use a focused editing surface with Preview, Timeline, Layers, Inspector and Assets views; bottom sheets/navigation; large touch targets; pinch zoom, two-finger pan and long-press menus. Do not squeeze desktop panels into a phone viewport. Horizontal timeline scrolling and vertical layer scrolling must remain independent. Degrade preview quality instead of blocking interaction.

## MCP and AI integration

Create a dedicated MCP server using the official TypeScript MCP SDK and versioned schemas shared with editor-core. MCP calls the same application services/commands as the UI.

### Transports and capabilities
Support stdio for local agents and Streamable HTTP for hosted remote use. Remote use requires authentication, tenant isolation, capability discovery, API versioning, pagination, structured errors, cancellation, progress, rate limits and audit logs. Expose resources, tools and workflow prompts. Keep MCP platform-neutral.

### Resources
- motionstudio://projects/{projectId}
- motionstudio://projects/{projectId}/compositions/{compositionId}
- motionstudio://projects/{projectId}/timeline
- motionstudio://projects/{projectId}/assets
- motionstudio://projects/{projectId}/render-jobs/{jobId}
- motionstudio://docs/project-schema
- motionstudio://docs/effects-catalog

### Tools

Read-only: list_projects, get_project, get_composition, get_timeline, list_layers, inspect_layer, list_assets, get_effect_catalog, get_render_job, validate_project, get_capabilities.

Editing: create_project, create_composition, add_layer, update_layer, remove_layer, reorder_layers, set_property, add_keyframe, move_keyframe, trim_layer, split_layer, add_effect, update_effect_parameters, add_marker, undo, redo.

Assets/output: request_asset_upload, relink_asset, create_proxy, preview_frame, validate_export_settings, start_render, cancel_render, download_render_artifact (short-lived authorized URL only).

### MCP security contract
- Authenticate every remote request; map identity to tenant and project permissions.
- Verify ownership on every call; never trust a model-supplied projectId.
- Separate project:read, project:edit, assets:write, render:create, export:download and admin scopes.
- Enforce rate limits, upload limits, render quotas, timeouts and payload caps.
- Use idempotency keys for mutations and render creation.
- Return stable structured error codes; never leak stack traces or secrets.
- Require explicit user confirmation for destructive edits, deletion, publishing, sharing and costly renders.
- Audit actor, action, project, time, request id and outcome; do not log media contents or credentials.
- Sandbox expressions and third-party effects. Never execute arbitrary agent-supplied JS in the server process.
- Use short-lived signed URLs, not permanent storage credentials.
- Test tenant isolation, IDOR, replay, injection and permission escalation.
- Never expose arbitrary shell, raw filesystem, unrestricted URL fetching or unbounded code execution.

### ChatGPT compatibility
MCP is the primary AI integration. Provide a separately documented OpenAPI adapter for compatible connectors/apps where appropriate. Do not target the retired legacy ChatGPT Plugins model or assume every ChatGPT account can install arbitrary plugins. Keep contracts platform-neutral.

## Security and privacy
Threat-model editor, uploads, MCP, collaboration, plugin SDK and render workers. Use secure sessions/OAuth, CSRF protection, CSP, secure cookies and strict CORS. Validate file signatures; decode in isolated workers; enforce archive limits and prevent zip bombs/path traversal. Keep secrets in managed secret storage and never commit .env or credentials. Make sharing opt-in/revocable. Respect fonts, stock media, codec and plugin licenses. Provide account export/deletion and AI privacy disclosures.

## Quality gates and acceptance criteria

Tests: unit tests for frame math, rational FPS, interpolation, transforms, snapping, command inverses and schemas; integration tests for import, relink, persistence, migrations, recovery, render jobs and MCP permissions; visual tests for text/shapes/alpha/blend/effects/responsive UI; E2E create → add layers → animate → preview → save → reload → export; performance tests for large timelines and mobile; security tests for tenant isolation, upload validation, scopes, confirmations and sandboxes.

First-release acceptance:
1. Create, save, reopen, duplicate, rename and delete projects with recovery.
2. Import supported image/video/audio assets and relink missing media.
3. Add text, vector, image and video layers.
4. Edit transforms and animate with frame-accurate keyframes.
5. Timeline supports trim, split, reorder, snapping, markers and undo/redo.
6. Preview and final render share scene semantics.
7. Export produces playable output with progress, cancellation and error recovery.
8. Projects and media survive browser restarts.
9. MCP read/edit tools use the same command layer and enforce scopes.
10. Every visible control is implemented and tested; no fake buttons.
11. Desktop and mobile E2E workflows pass on supported browsers.
12. A clean clone installs and runs using documented commands.

## Roadmap

- Phase 0 Foundation: monorepo, CI, lint/typecheck/test, design tokens, project schema, ADRs, secure config, local setup.
- Phase 1 Editing slice: dashboard, compositions, layers, timeline, text/shape/image, transforms, keyframes, undo/redo, IndexedDB.
- Phase 2 Renderer: scene graph, GPU preview, effect graph, decode, frame cache, deterministic render tests.
- Phase 3 Media/export: asset library, proxies, waveform/audio, render workers, export queue, image sequences and supported video formats.
- Phase 4 Professional tools: graph editor, masks, parenting, precomps, blend modes, text animator, color management, motion blur and 3D foundations.
- Phase 5 Cloud: auth, database, object storage, versions, sync, presence, permissions, comments and conflict resolution.
- Phase 6 MCP/AI: resources, tools, prompts, OAuth, scopes, approval gates, audit, examples and evaluations.
- Phase 7 Scale: plugin SDK, render farm, advanced effects, templates, quotas, telemetry, performance and security hardening.

Each phase ends with a working demo, tests, docs and a tagged release. Do not skip foundations for a flashy prototype.

## Developer commands

Expected commands after scaffold implementation:

    corepack enable
    pnpm install
    pnpm dev
    pnpm lint
    pnpm typecheck
    pnpm test
    pnpm test:e2e
    pnpm build

Create .env.example with placeholders only. Provide a local development mode without paid services where practical.

## AI coding rules

Read AGENT_PROMPT.md before coding. Inspect the whole repository and report its state. Make a file-level plan with risks. Work in small compiling increments. Do not replace working functionality with mockups, add dead buttons/fake progress, or claim completion without code/tests/user workflow. Keep TypeScript strict, avoid any, validate untrusted inputs, test each feature, run all relevant checks, update docs/ADRs, preserve user data, provide migrations, and never commit secrets. Prefer open-source/free services where feasible without sacrificing security. If scope is too large, ship a complete vertical slice and list deferred work.

## Current repository status

This repository is being initialized as a professional blueprint and starter structure. The previous browser prototype deployed separately is not automatically a complete source repository and must not be represented as production-ready. The next coding agent must scaffold the monorepo and implement the roadmap incrementally.

## Start here
1. Read AGENT_PROMPT.md.
2. Read docs/architecture/OVERVIEW.md.
3. Build and verify Phase 0.
4. Deliver the first real editing vertical slice before expanding scope.
