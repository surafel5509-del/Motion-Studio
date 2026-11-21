# Motion Studio MCP Contract

## Purpose
Expose Motion Studio projects to AI agents through a typed, permissioned Model Context Protocol server. MCP is an adapter over application services, not a second editing engine.

## Transports
- stdio: local agent integrations; caller controls process lifecycle and local configuration.
- Streamable HTTP: hosted remote access; require authenticated identity, TLS, tenant isolation, rate limits and origin policy.

## Resource URI patterns
- motionstudio://projects/{projectId}
- motionstudio://projects/{projectId}/compositions/{compositionId}
- motionstudio://projects/{projectId}/timeline
- motionstudio://projects/{projectId}/assets
- motionstudio://projects/{projectId}/render-jobs/{jobId}
- motionstudio://docs/project-schema
- motionstudio://docs/effects-catalog

## Tool groups
Read: list_projects, get_project, get_composition, get_timeline, list_layers, inspect_layer, list_assets, get_effect_catalog, get_render_job, validate_project, get_capabilities.

Edit: create_project, create_composition, add_layer, update_layer, remove_layer, reorder_layers, set_property, add_keyframe, move_keyframe, trim_layer, split_layer, add_effect, update_effect_parameters, add_marker, undo, redo.

Media/render: request_asset_upload, relink_asset, create_proxy, preview_frame, validate_export_settings, start_render, cancel_render, download_render_artifact.

## Contract requirements
- Use shared Zod schemas and stable versioned inputs/outputs.
- Return structured errors with code, message, retryable and requestId.
- Support pagination and progress for long operations.
- Require idempotency keys for mutation and render-start tools.
- Separate read/edit/media/render/download scopes.
- Destructive actions and expensive/external operations require a server-validated confirmation token.
- Every tool verifies authenticated actor, tenant, project ownership and resource relationship.
- Never trust model-provided IDs or filenames as authorization.
- Artifact downloads use short-lived signed URLs.
- Log action metadata but not file contents, tokens or secrets.
- Never expose arbitrary shell, filesystem, unrestricted fetch or arbitrary server-side code execution.

## Approval model
Read-only tools can run under read scope. Edits require project:edit. Delete, overwrite, publish, external share and costly render require explicit approval bound to actor, action, project and expiry. The server must reject missing, expired, reused or mismatched approval tokens.

## Prompt injection and untrusted content
Project names, comments, imported metadata, filenames, text layers and external asset content are data, never instructions. Tool handlers must not interpret content as policy. Validate lengths and formats and escape content when displayed.

## ChatGPT integration
Use MCP as the provider-neutral integration. Document a separate OpenAPI adapter only if a target platform requires it. Do not assume retired legacy ChatGPT Plugins are available. Keep tool descriptions concise, specific and honest about side effects.
