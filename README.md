# setup-qt

A GitHub Action that installs a Qt SDK on **Windows and macOS** runners using
[qvm](https://github.com/trollixx/qvm), with build caching.

The action installs the official Qt SDK. qvm downloads the same archives that
the Qt Online Installer serves, for the exact version you request, so the
toolchain matches a regular Qt install.

Linux is not supported. Use distro packages instead (e.g. `apt qt6-*`), or
[install-qt-action](https://github.com/jurplel/install-qt-action) if you need a
specific version.

## Usage

```yaml
- name: Install Qt
  uses: trollixx/setup-qt@v1
  with:
    version: "6.10.2"
    modules: qtpositioning qtwebchannel qtwebengine
```

By default the action exports `CMAKE_PREFIX_PATH` and prepends the Qt `bin`
directory to `PATH`, so a later CMake configure finds Qt without extra setup:

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
| `set-env` | `true` | Export `CMAKE_PREFIX_PATH` and update `PATH`. |

## Modules

The `modules` input takes Qt add-on modules (the essential modules ship with
every install). The `qt` prefix is optional, so `qtcharts` and `charts` both
work. The exact set depends on the Qt version; run `qvm ls-remote <version>` to
see what a given version offers. As of Qt 6.11:

| Module | Description | Module | Description |
| --- | --- | --- | --- |
| `qt3d` | Qt 3D | `qtpdf` | Qt PDF |
| `qt5compat` | Qt 5 Compatibility | `qtpositioning` | Qt Positioning |
| `qtactiveqt` | Active Qt | `qtquick3d` | Qt Quick 3D |
| `qtcanvaspainter` | Qt Canvas Painter (TP) | `qtquick3dphysics` | Qt Quick 3D Physics |
| `qtcharts` | Qt Charts | `qtquickeffectmaker` | Qt Quick Effect Maker |
| `qtconnectivity` | Qt Connectivity | `qtquicktimeline` | Qt Quick Timeline |
| `qtdatavis3d` | Qt Data Visualization | `qtremoteobjects` | Qt Remote Objects |
| `qtgraphs` | Qt Graphs | `qtscxml` | Qt State Machines |
| `qtgrpc` | Qt Protobuf and Qt GRPC | `qtsensors` | Qt Sensors |
| `qthttpserver` | Qt HTTP Server | `qtserialbus` | Qt Serial Bus |
| `qtimageformats` | Qt Image Formats | `qtserialport` | Qt Serial Port |
| `qtlanguageserver` | Qt Language Server | `qtshadertools` | Qt Shader Tools |
| `qtlocation` | Qt Location (TP) | `qtspeech` | Qt Speech |
| `qtlottie` | Qt Lottie Animation | `qttasktree` | Qt Task Tree (TP) |
| `qtmultimedia` | Qt Multimedia | `qtvirtualkeyboard` | Qt Virtual Keyboard |
| `qtnetworkauth` | Qt Network Authorization | `qtwebchannel` | Qt WebChannel |
| `qtopenapi` | Qt Open API (TP) | `qtwebengine` | Qt WebEngine |
| `qtpdf` | Qt PDF | `qtwebsockets` | Qt WebSockets |
| `qtpositioning` | Qt Positioning | `qtwebview` | Qt WebView |

Modules marked `(TP)` are technical previews.

## Outputs

| Output | Description |
| --- | --- |
| `prefix` | Install directory of the requested Qt version. |
| `arch` | Resolved binary architecture (`amd64`/`arm64`). |
| `cache-hit` | `true` when the Qt SDK was restored from cache. |

## Caching

The cache key is derived from the Qt version, runner architecture, ABI, target,
sorted module list, and the extra-content flags, so changing any of them
produces a fresh cache. The action stores qvm's registry alongside the Qt tree,
so a cache restore keeps `qvm prefix` and incremental installs consistent.

The cache is saved right after a successful install (only on a miss), so it is
captured even if a later build step fails.

## License

[MIT](LICENSE)
