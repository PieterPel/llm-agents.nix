## Summary

Adds opt-in support for using system-provided ONNX Runtime bindings and fixes embeddings in sandboxed environments by ensuring a writable Transformers.js cache directory.

## Motivation / context

Packaging in sandboxed environments needs a way to use a prebuilt `onnxruntime_binding.node` without disabling features. Embeddings were also failing because Transformers.js tried to write its cache under a read-only install path. Related to issue #376.

## Areas touched

- [x] `gitnexus/` (CLI / core / MCP server)
- [ ] `gitnexus-web/` (Vite / React UI)
- [ ] `.github/` (workflows, actions)
- [ ] `eval/` or other tooling
- [x] Docs / agent config only (`AGENTS.md`, `CLAUDE.md`, `.cursor/`, `llms.txt`, etc.)

## Scope & constraints

**In scope**

- Add loader that redirects `onnxruntime_binding.node` to a user-specified path.
- Wire the override into both core embedder entrypoints.
- Configure Transformers.js cache dir to a writable location for sandboxed installs.
- Document the environment variable and usage.

**Explicitly out of scope / not done here**

- Changing any embedding logic or model behavior.
- Fixing the LadybugDB native binary issue (#376).
- Adjusting test expectations in CI.

## Implementation notes

- The override hooks Node’s module resolution so that requests for `onnxruntime_binding.node` resolve to the path in `GITNEXUS_ORT_BINDING_PATH`.
- Transformers.js cache is set to `$HOME/.cache/gitnexus/transformers` when possible, avoiding writes to read-only install paths.
- Both changes are opt‑in / safe defaults: the binding override only applies if the env var is set; cache setup falls back silently if not writable.

## Testing & verification

- [x] `cd gitnexus && npm test` (via docker; see note)
- [ ] `cd gitnexus && npm run test:integration` *(not needed here)*
- [x] `cd gitnexus && npx tsc --noEmit` (via docker)
- [ ] `cd gitnexus-web && npm test` *(not applicable)*
- [ ] `cd gitnexus-web && npx tsc -b --noEmit` *(not applicable)*
- [x] Manual: `gitnexus analyze --embeddings .` (succeeds in sandboxed install)

**Notes:**\
Ran in Linux amd64 container (`node:20-trixie`) because local macOS tests are blocked by #376. One unit test fails in container due to root permissions (`test/unit/ignore-service.test.ts` “warns on EACCES but does not throw”); all other tests pass.

Commands:

```
docker run --rm --platform=linux/amd64 -v /Users/pieterpel/home/private-projects/llm-agents.nix/GitNexus:/app -w /app/gitnexus node:20-trixie bash -lc "apt-get update && apt-get install -y python3 make g++ build-essential pkg-config git && npm ci"
docker run --rm --platform=linux/amd64 -v /Users/pieterpel/home/private-projects/llm-agents.nix/GitNexus:/app -w /app/gitnexus node:20-trixie bash -lc "npx tsc --noEmit && npm test"
```

## Risk & rollout

- Low risk; opt‑in only. No migration needed.
- Does not affect default behavior unless `GITNEXUS_ORT_BINDING_PATH` is set.

## Checklist

- [x] PR body meets repo minimum length (workflow may label short descriptions)
- [x] If `AGENTS.md` / overlays changed: headers, scope block, and changelog updated per project conventions
- [x] No secrets, tokens, or machine-specific paths committed

EDIT: Also added cache-dir configuration to avoid read-only install paths.
