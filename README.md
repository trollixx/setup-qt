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
| `qt3d` | Qt 3D | `qtpositioning` | Qt Positioning |
| `qt5compat` | Qt 5 Compatibility | `qtquick3d` | Qt Quick 3D |
| `qtactiveqt` | Active Qt | `qtquick3dphysics` | Qt Quick 3D Physics |
| `qtcanvaspainter` | Qt Canvas Painter (TP) | `qtquickeffectmaker` | Qt Quick Effect Maker |
| `qtcharts` | Qt Charts | `qtquicktimeline` | Qt Quick Timeline |
| `qtconnectivity` | Qt Connectivity | `qtremoteobjects` | Qt Remote Objects |
| `qtdatavis3d` | Qt Data Visualization | `qtscxml` | Qt State Machines |
| `qtgraphs` | Qt Graphs | `qtsensors` | Qt Sensors |
| `qtgrpc` | Qt Protobuf and Qt GRPC | `qtserialbus` | Qt Serial Bus |
| `qthttpserver` | Qt HTTP Server | `qtserialport` | Qt Serial Port |
| `qtimageformats` | Qt Image Formats | `qtshadertools` | Qt Shader Tools |
| `qtlanguageserver` | Qt Language Server | `qtspeech` | Qt Speech |
| `qtlocation` | Qt Location (TP) | `qttasktree` | Qt Task Tree (TP) |
| `qtlottie` | Qt Lottie Animation | `qtvirtualkeyboard` | Qt Virtual Keyboard |
| `qtmultimedia` | Qt Multimedia | `qtwebchannel` | Qt WebChannel |
| `qtnetworkauth` | Qt Network Authorization | `qtwebengine` | Qt WebEngine |
| `qtopenapi` | Qt Open API (TP) | `qtwebsockets` | Qt WebSockets |
| `qtpdf` | Qt PDF | `qtwebview` | Qt WebView |

Modules marked `(TP)` are technical previews.

Modules that the requested modules depend on are installed automatically. For
example, `qthttpserver` pulls in `qtwebsockets`, and `qtquick3d` pulls in
`qtshadertools` and `qtquicktimeline`.

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
