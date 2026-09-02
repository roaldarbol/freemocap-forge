# freemocap-forge

Conda packages for the [FreeMoCap](https://freemocap.org) **v2 alpha** stack,
built with [rattler-build](https://rattler.build) and published to
[https://prefix.dev/freemocap-forge](https://prefix.dev/freemocap-forge).

```sh
pixi add -c https://prefix.dev/freemocap-forge -c conda-forge freemocap
# or
conda install -c https://prefix.dev/freemocap-forge -c conda-forge freemocap
```

Platforms: `linux-64`, `osx-arm64`, `win-64` (no `osx-64` — upstream ships no
mediapipe wheel for Intel macs). Python 3.11–3.12.

## Packages

| Recipe | Version | Source |
| --- | --- | --- |
| `freemocap` | 2.0.0a23 | tag `v2.0.0-alpha.23` |
| `skellycam` | 2.0.0a6 | `skellycam` @ main (commit-pinned) |
| `skellytracker` | 2024.9.1019 | `skellytracker` @ main (commit-pinned); equals `skellytracker[all-cpu]` |
| `skellyforge` | 2024.12.1009 | `skellyforge` @ main (commit-pinned) |
| `skelly-synchronize` | 2025.4.1037 | `skellysync` @ `philip/rewrite` (commit-pinned) |
| `skellylogs` | 0.1.0 | `skellylogs` @ main (commit-pinned) |
| `skellypings` | 0.1.0 | `skellypings` @ main (commit-pinned) |
| `freemocap-blender-addon` | 2026.4.1041 | `freemocap_blender_addon` @ main (commit-pinned) |
| `mediapipe` | 0.10.33 | repackaged PyPI wheels (not on conda-forge) |

Upstream resolves the sub-packages from git branches via `uv` sources; each
recipe here pins the corresponding commit in `context.git_rev`. Everything else
comes from conda-forge. GPU tracker backends (CUDA/TensorRT/DirectML) are
pip-only extras upstream and are not packaged — `skellytracker` here matches
the `all-cpu` extra.

## How CI works

One workflow, no per-recipe matrix, no change detection:

- Every run calls `rattler-build` with `--skip-existing=all` over the recipes.
  Anything already published (same version + build number) is skipped;
  the rest is built in dependency order (rattler-build sorts recipes
  topologically), so even an empty channel bootstraps in a single run.
- The `linux-64` job builds **all** recipes (it owns the `noarch` ones);
  `osx-arm64` and `win-64` build only the platform-specific recipes
  (currently just `mediapipe` — see the `build-arch` task in `pixi.toml`).
- Pull requests build without uploading; pushes to `main` and manual
  `workflow_dispatch` runs upload to prefix.dev.

## Updating a package

1. Find the new commit: `git ls-remote https://github.com/freemocap/<repo> <branch>`
2. In the recipe, update `context.git_rev` (and `context.version` if the
   upstream `__version__` / `pyproject.toml` version changed; otherwise bump
   `build.number`).
3. Open a PR — CI builds exactly the recipes whose version/build changed.

For `freemocap` itself, update `context.tag`/`context.version` and the tarball
`sha256` (`curl -sL <url> | shasum -a 256`).

## Local builds

```sh
pixi run build                     # everything not already on the channel
pixi run build-arch                # platform-specific recipes only
pixi run rattler-build build --recipe recipes/<name> -c https://prefix.dev/freemocap-forge -c conda-forge
```

## Publishing setup (one-time)

1. Create the `freemocap-forge` channel on [prefix.dev](https://prefix.dev).
2. Configure this GitHub repository as a **trusted publisher** for the channel
   (channel settings → Trusted Publishers); the workflow authenticates via
   OIDC, no API key secret needed.
3. Run the workflow manually (`workflow_dispatch`) to bootstrap the channel.

To publish from your machine instead: `rattler-build upload prefix -c freemocap-forge output/**/*.conda`
with a `PREFIX_API_KEY` in the environment.

The channel name is referenced in the `pixi.toml` tasks — change it there if
you deploy elsewhere.
