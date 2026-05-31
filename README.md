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

By default the action exports `CMAKE_PREFIX_PATH` and `Qt6_DIR`, and prepends the
Qt `bin` directory to `PATH`, so a later CMake configure finds Qt without extra
setup:

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

## Modules

The `modules` input takes Qt add-on modules (the essential modules ship with
every install). The `qt` prefix is optional, so `qtcharts` and `charts` both
work. The exact set depends on the Qt version; run `qvm ls-remote <version>` to
see what a given version offers. As of Qt 6.11:

| Module | Description | Module | Description |
| --- | --- | --- | --- |
| `qt3d` | Qt 3D | `qtopenapi` | Qt Open API (TP) |
| `qt5compat` | Qt 5 Compatibility | `qtpdf` | Qt PDF |
| `qtactiveqt` | Active Qt | `qtpositioning` | Qt Positioning |
| `qtcanvaspainter` | Qt Canvas Painter (TP) | `qtquick3d` | Qt Quick 3D |
| `qtcharts` | Qt Charts | `qtquick3dphysics` | Qt Quick 3D Physics |
| `qtconnectivity` | Qt Connectivity | `qtquickeffectmaker` | Qt Quick Effect Maker |
| `qtdatavis3d` | Qt Data Visualization | `qtquicktimeline` | Qt Quick Timeline |
| `qtgraphs` | Qt Graphs | `qtremoteobjects` | Qt Remote Objects |
| `qtgrpc` | Qt Protobuf and Qt GRPC | `qtscxml` | Qt State Machines |
| `qthttpserver` | Qt HTTP Server | `qtsensors` | Qt Sensors |
| `qtimageformats` | Qt Image Formats | `qtserialbus` | Qt Serial Bus |
| `qtlanguageserver` | Qt Language Server | `qtserialport` | Qt Serial Port |
| `qtlocation` | Qt Location (TP) | `qtshadertools` | Qt Shader Tools |
| `qtlottie` | Qt Lottie Animation | `qtspeech` | Qt Speech |
| `qtmultimedia` | Qt Multimedia | `qttasktree` | Qt Task Tree (TP) |
| `qtnetworkauth` | Qt Network Authorization | `qtvirtualkeyboard` | Qt Virtual Keyboard |
| `qtwebchannel` | Qt WebChannel | `qtwebengine` | Qt WebEngine |
| `qtwebsockets` | Qt WebSockets | `qtwebview` | Qt WebView |

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
