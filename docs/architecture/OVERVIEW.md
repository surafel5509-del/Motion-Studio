# Motion Studio Architecture Overview

## Context
Motion Studio is a browser-based motion graphics editor with desktop and mobile interaction models. It needs a real timeline and renderer, not a UI-only simulation. This document describes intended boundaries; update it through ADRs as implementation choices are validated.

## High-level flow

    Browser UI
      -> Editor application services
        -> Typed command bus
          -> Versioned project document
          -> Undo/redo history
          -> Autosave/change log
        -> Scene evaluator
          -> Timeline and animation evaluation
          -> Scene graph and effect graph
            -> WebGL2/WebGPU preview backend
            -> Headless render backend
        -> Media service
          -> Asset metadata and local IndexedDB
          -> Decode, thumbnail, proxy and waveform workers
        -> Export orchestration
          -> Render job queue
          -> Render worker and encoder
          -> Artifact storage

    MCP clients
      -> MCP transport and authentication
        -> MCP tools/resources/prompts
          -> Same editor application services and command bus
            -> Same project validation, permissions, audit and persistence

## Architectural rules
1. Project schema is canonical; UI component state is not the document.
2. All edits use commands with validation, permission checks, inverse operations and change events.
3. Renderer code does not import React.
4. Timeline and animation math are pure TypeScript and use integer frame coordinates.
5. Frame rate is rational, not a floating-point approximation.
6. Preview and final render share scene evaluation and effect semantics.
7. MCP never writes raw project JSON; it invokes application commands.
8. Cloud, collaboration and plugin features are optional adapters around stable core contracts.
9. Media decoding and untrusted plugin execution occur in isolated workers/processes.
10. Every public API and persisted document is versioned.

## Core package responsibilities
- project-schema: schemas, migrations, serialization, import validation.
- editor-core: command bus, history, selection, transactions and application services.
- timeline: clip math, snapping, track layout and timeline interactions.
- animation: keyframe models, interpolation, easing and expression sandbox interface.
- scene-graph: node graph, transforms, parenting, compositing order.
- renderer: backend interfaces, GPU resources, render graph and cache.
- effects: effect manifests, parameter schemas and built-in effect implementations.
- media: import validation, metadata, thumbnails, decode adapters, proxies and relinking.
- audio: Web Audio graph, waveform generation, timing and meters.
- export: render plans, queue contracts, encoders and artifact metadata.
- mcp-contracts: versioned MCP input/output schemas and permission definitions.

## Project document invariants
- schemaVersion is required and migrations are explicit.
- All entities use stable IDs.
- Timeline positions and durations are integer frame counts.
- FPS is represented as numerator/denominator.
- Assets are references with content hashes, not embedded huge binary blobs.
- Unknown required feature versions fail safely; optional unknown metadata can be preserved.
- Every document is validated before loading and after migration.

## Performance model
Keep UI input responsive. Move expensive decode, waveform, thumbnail, render and expression work off the main thread. Use cancellation and backpressure. Bound GPU and CPU caches. Use preview quality adaptation under load. Profile on mid-range mobile devices, not only developer desktops.

## Failure and recovery
Autosave should persist a recoverable operation log and periodic project snapshots. A failed render must not corrupt the editable project. Device/GPU loss should restore renderer resources from the project document. Asset loss should mark relink-required state rather than silently substituting files.

## Open architecture decisions
Create ADRs before committing to:
- renderer backend and scene graph library;
- database ORM and cloud storage provider;
- collaboration consistency model;
- plugin isolation/runtime;
- expression language and sandbox;
- export/encoding deployment topology.
