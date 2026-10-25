# OpenUp-deps

Prebuilt dependencies for [OpenUp](https://github.com/ivlab/OpenUp), so a
fresh machine (a Claude Code cloud session, a CI runner, a lab desktop)
can build OpenUp and make its render goldens without building them first.
Building VTK takes an hour or more on every system it is needed on.

Today this holds one build, VTK 9.4.2 for Ubuntu 24.04 on x86_64. The
layout and the names leave room for Windows and macOS builds, more
architectures, and more VTK versions beside it.

## Layout

```
builds.json                     every build: its settings, file and sha256
vtk/
  licenses/                     VTK's own license
  <vtk version>/
    <build name>.<ext>          the build, an install prefix
    <build name>.<ext>.sha256   its sha256, as `sha256sum` writes it
```

## Build names

```
vtk-<version>-<profile>-<os>-<os version>-<arch>-<toolchain>-<config>.<ext>
```

Each field is something two builds can differ in and be incompatible:
take the build whose every field matches your machine and OpenUp
checkout. Fields are lower case, with no hyphens inside one.

| field | what it is | values so far, and what is planned |
|---|---|---|
| `version` | VTK's release | `9.4.2` |
| `profile` | the module list and CMake options, as OpenUp needs them (below); a new profile when they change | `openup1` |
| `os` | the operating system | `linux`, then `macos`, `windows` |
| `os version` | the oldest system it runs on: the distribution and release on Linux (its glibc and libstdc++), the deployment target on macOS, the oldest Windows | `ubuntu24.04`; `macos13`; `win10` |
| `arch` | the CPU | `x86_64`; `arm64`; `universal` (macOS fat binaries) |
| `toolchain` | the compiler and the C++ runtime it links, which must match what OpenUp is built with | `gcc13` (libstdc++); `appleclang15` (libc++); `msvc143` (Visual Studio 2022, `/MD`) |
| `config` | the build type; on Windows a Debug OpenUp needs a Debug VTK, because their C runtimes differ | `release`; `debug`; `relwithdebinfo` |
| `ext` | the archive | `tar.gz` on Linux and macOS; `zip` on Windows |

For example:

```
vtk-9.4.2-openup1-linux-ubuntu24.04-x86_64-gcc13-release.tar.gz   (here now)
vtk-9.4.2-openup1-macos-macos13-arm64-appleclang15-release.tar.gz (planned)
vtk-9.4.2-openup1-windows-win10-x86_64-msvc143-release.zip        (planned)
vtk-9.4.2-openup1-windows-win10-x86_64-msvc143-debug.zip          (planned)
```

## Profiles

A profile names everything about a build's configuration besides the
platform fields. Two builds of one profile on different platforms offer
OpenUp the same VTK.

### `openup1`

What OpenUp's `openup_vtk` (`src/openup/vtk/CMakeLists.txt`) and its
examples 05 and 10 ask for, and nothing else. It is the configuration in
OpenUp's `tools/setup_golden_env.sh`.

- Modules: CommonCore, CommonDataModel, FiltersCore, FiltersGeneral,
  IOImage, IOPLY, ImagingHybrid, RenderingCore, RenderingOpenGL2,
  RenderingVolumeOpenGL2, RenderingExternal, and what they depend on.
  Every other group is off: no Qt, MPI, Views or Web.
- `BUILD_SHARED_LIBS=ON`; no Python wrapping, no tests, no examples.
- **nlohmann json external** (`VTK_MODULE_USE_EXTERNAL_VTK_nlohmannjson=ON`):
  VTK uses the system's json, not its own copy, so VTK's copy never meets
  OpenUp's in one translation unit. A build with VTK's vendored json
  cannot share a file with OpenUp, so it is a different profile. On
  Windows and macOS, build against the same nlohmann json version OpenUp
  fetches (v3.11.3).
- OpenGL windowing is the platform's own: X11 with EGL on Linux, Cocoa on
  macOS, WGL on Windows.

## The builds

`builds.json` lists each build with every field above, the settings
behind it, its file, its size and its sha256. Use it to choose a
download, and check the sha256 before unpacking.

### `vtk-9.4.2-openup1-linux-ubuntu24.04-x86_64-gcc13-release.tar.gz`

Built in a Claude Code cloud container (Ubuntu 24.04, glibc 2.39, GCC
13.3) by `tools/setup_golden_env.sh` from VTK's `v9.4.2` tag. OpenGL is
X11 with EGL. It is an install prefix: `tar xzf` it into `$HOME` and it
becomes `~/vtk-install`.

- sha256 `b36644aa8271ecfe83e62ba94f35f87e9c620cdf314315e4be54e4a64cbede9d`
- About 62 MB packed, 253 MB unpacked.

## Adding a build

1. Build VTK with the profile's options (for `openup1`, those in
   `tools/setup_golden_env.sh`), installed to a prefix named
   `vtk-install`.
2. Pack the prefix: `tar czf <name>.tar.gz -C <parent> vtk-install`, or
   on Windows a `.zip` whose top folder is `vtk-install`.
3. Name it by the convention above, put it in `vtk/<version>/`, and
   write `sha256sum <name> > <name>.sha256`.
4. Add its entry to `builds.json`, with anything about how it was built
   that the fields do not say.
5. A file must be under 100 MB to be committed. A larger build (a Debug
   build on Windows, a macOS universal build) goes in a release of this
   repository as an asset under the same name, and its `builds.json`
   entry gives that URL.

## Licenses

VTK is © Ken Martin, Will Schroeder and Bill Lorensen, under the BSD
3-clause license in `vtk/licenses/VTK-Copyright.txt`. The licenses of the
third-party libraries VTK vendors are inside each build, under
`vtk-install/share/licenses/VTK/`. Nothing here is OpenUp's own code.
