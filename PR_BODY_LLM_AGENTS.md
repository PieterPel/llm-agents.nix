## Summary

Adds a GitNexus package with a temporary patch for system ONNX Runtime binding + writable Transformers.js cache so embeddings work in sandboxed installs while upstream PR is pending.

## Motivation / context

GitNexus currently fails in sandboxed environments due to read‑only cache paths and requires a system ONNX Runtime binding override. Upstream fixes are in progress; this PR applies them locally so the package is usable now.

## Areas touched

- [x] `packages/gitnexus/`
- [ ] `scripts/`
- [ ] `docs/`

## Scope & constraints

**In scope**

- Package GitNexus with build fixes and runtime defaults.
- Apply patch containing upstream fixes until merged.

**Explicitly out of scope / not done here**

- Any upstream code changes beyond the patch already carried.

## Implementation notes

- Uses upstream `GitNexus` source with a patch (from `packages/gitnexus/system-onnxruntime-node.patch`).
- Ensures `gitnexus-shared` is built at package time.
- Copies LadybugDB native binding into `@ladybugdb/core` for runtime.
- Wraps `gitnexus` to set cache env vars and `GITNEXUS_ORT_BINDING_PATH`.

## Testing & verification

- [x] `nix fmt`
- [x] `nix run --accept-flake-config nixpkgs#python311 -- ./scripts/generate-package-docs.py` (no README changes)
- [x] `nix build --accept-flake-config .#gitnexus`
- [ ] `nix flake check` (fails: `zeroclaw` source URL 404 during checks on Apple silicon)

## Risk & rollout

- Low risk; patch should be removed once upstream merges.

## Checklist

- [x] No secrets, tokens, or machine-specific paths committed
