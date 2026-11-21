# Contributing

Motion Studio is a long-term engineering project. Prefer small, tested vertical slices over broad incomplete scaffolding.

## Workflow
1. Read README.md and AGENT_PROMPT.md.
2. Open an issue or write a short plan for substantial changes.
3. Create a focused branch.
4. Keep UI, project model, commands, rendering and MCP boundaries intact.
5. Add tests and update docs/ADRs.
6. Run lint, typecheck, unit tests, relevant integration/E2E tests and build.
7. Submit a PR describing behavior, test evidence, screenshots/video for UI changes, risks and deferred work.

## Standards
- TypeScript strict mode; no avoidable any.
- Validate data at boundaries with shared schemas.
- Use integer frames and rational frame rates.
- Do not commit credentials, media files, copyrighted assets or generated build output.
- Do not add controls that do nothing.
- Document browser/runtime compatibility and codec limitations.
- Respect third-party licenses and security boundaries.
- MCP mutations must use application services and permission checks.

## Commit style
Use conventional prefixes such as feat, fix, docs, refactor, test, chore and security.
