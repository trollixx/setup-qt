# setup-qt

A GitHub Action that installs a Qt SDK on **Windows** runners using
[qvm](https://github.com/trollixx/qvm), with build caching.

For Linux/macOS use distro packages or
[`jurplel/install-qt-action`](https://github.com/jurplel/install-qt-action).
This action is Windows-only because qvm publishes win64 binaries (amd64 + arm64).

## Usage

```yaml
- name: Install Qt
  uses: trollixx/setup-qt@v1
  with:
    version: "6.10.2"
    modules: qtpositioning qtwebchannel qtwebengine
    cache: true
```

By default the action exports `CMAKE_PREFIX_PATH` and `Qt6_DIR`, and prepends the
Qt `bin` directory to `PATH`, so a subsequent CMake configure finds Qt with no
extra wiring:

```yaml
- uses: trollixx/setup-qt@v1
  with:
    version: "6.10.2"
- run: cmake -B build -G Ninja   # CMAKE_PREFIX_PATH already points at Qt
```

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `version` | *(required)* | Qt version to install (e.g. `6.10.2`). |
| `modules` | - | Add-on modules, space- or comma-separated (`qt` prefix optional). |
| `arch` | auto | Compiler/ABI target (e.g. `win64_msvc2022_64`). Auto-detected if empty. |
| `target` | `desktop` | Qt platform: `desktop`, `android`, `ios`, `wasm`. |
| `docs` / `examples` / `sources` / `debug-symbols` | `false` | Install extra content. |
| `dir` | runner temp | Qt install root. |
| `cache` | `true` | Cache the installed Qt SDK across runs. |
| `cache-key-prefix` | `qvm` | Cache key prefix (bump to invalidate caches). |
| `qvm-version` | pinned | qvm release tag to download. |
| `set-env` | `true` | Export `CMAKE_PREFIX_PATH`, `Qt6_DIR`, and update `PATH`. |

## Outputs

| Output | Description |
| --- | --- |
| `prefix` | Install directory of the requested Qt version. |
| `arch` | Resolved binary architecture (`amd64`/`arm64`). |
| `cache-hit` | `true` when the Qt SDK was restored from cache. |

## Caching

The cache key is derived from the Qt version, runner architecture, ABI, target,
sorted module list, and the extra-content flags — so changing any of them
produces a fresh cache. The action stores qvm's registry alongside the Qt tree,
so a cache restore keeps `qvm prefix` and incremental installs consistent.

The cache is saved immediately after a successful install (only on a miss), so
it is captured even if a later build step fails.

## License

[MIT](LICENSE)
