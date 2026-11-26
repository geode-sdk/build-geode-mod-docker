# build-geode-mod-docker

GitHub action to build mods using [geode-docker](https://github.com/geode-sdk/geode-docker) images.

> [!WARNING]
> To build macOS as split `macm1` and `macintel`, you need to remove `set(CMAKE_OSX_ARCHITECTURES ...)` from your CMakeLists.txt

> [!NOTE]
> Missing `cli` version override, but otherwise feature complete with [build-geode-mod](https://github.com/geode-sdk/build-geode-mod)

## Example

```yaml
name: Build Geode Mod

on:
  push:

jobs:
  build:
    strategy:
      fail-fast: false
      matrix:
        config:
        - target: win
        # see warning above
        - target: macm1
        - target: macintel
        # this works too
        # - target: mac
        - target: ios
        - target: android32
        - target: android64

    name: Build ${{ matrix.config.target }}
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v7

      - uses: geode-sdk/build-geode-mod-docker@v1
        with:
          target: ${{ matrix.config.target }}
          upload: true

  merge:
    name: Merge
    runs-on: ubuntu-latest
    needs: ['build']

    steps:
      - uses: geode-sdk/build-geode-mod-docker/merge@v1
        with:
          upload: true
```

# Options

## Build action

```yml
target:
  description: One of win, android32, android64, mac, macintel, macm1, ios.
  required: true

sdk:
  description: Which version of the SDK to use. Set to 'given' to use the version specified in mod.json, 'nightly' for latest commit, 'latest' for latest release.
  default: given

configure-args:
  description: Extra arguments passed to CMake when configuring.
  default: ""

build-args:
  description: Extra arguments passed to CMake when building.
  default: ""

build-config:
  description: Which build configuration to use, can only be Release or RelWithDebInfo. Defaults to Release.
  default: Release

path:
  description: Path to the project which to build. Defaults to current directory.
  default: ./

upload:
  description: Automatically upload built files as geode-build-(platform).zip
  default: false

bindings:
  description: What repository to use for bindings. Must be in the format "user/repo".
  default: "geode-sdk/bindings"

bindings-ref:
  description: Which commit/branch to use for bindings. Defaults to latest commit in main branch.
  default: ""

use-action-cache:
  description: Whether to use actions cache for CPM and sccache.
  default: true

save-action-cache:
  description: Whether to save actions cache, can be useful to disable on PRs and such.
  default: true

use-lto:
  description: Whether to use LTO (Link Time Optimization) via the CMAKE_INTERPROCEDURAL_OPTIMIZATION flag, improving build size. Defaults to true.
  default: true

use-sccache:
  description: Whether to use sccache for builds. Defaults to true.
  default: true

sccache-cache-size:
  description: Max size for sccache cache. Defaults to 1G.
  default: 1G

export-debug:
  description: Whether to export debug files for builds (.pdb for Windows, .dbg for Android, .dSYM for MacOS). Defaults to false.
  default: false

export-symbols:
  description: Whether to export Breakpad symbols for Android builds. Defaults to false.
  default: false

export-pdb:
  description: Whether to export PDB files for Windows builds. Defaults to false.
  default: false

bundle-pdb:
  description: Whether to bundle PDB files into the .geode package. Requires SDK v4.2.0 and CLI v3.3.0 at minimum. Defaults to false.
  default: false

parallel:
  description: How many parallel jobs to use. 0 will disable, empty string will use cmake's "--parallel".
  default: ""
```

## Merge action

```yml
cli:
  description: Which CLI version to use, defaults to latest release.
  default: latest

delete-artifacts:
  description: Whether to delete temporary artifacts left by the build action.
  default: true

zstd:
  description: Whether to use zstd compression when merging the output .geode files.
  default: true

optimize-pngs:
  description: Whether to recompress PNGs during merging to reduce file size.
  default: true

upload:
  description: Upload the artifact
  default: false

upload-zip:
  description: Forces the uploaded artifact to be a zip even if theres only one .geode file.
  default: false
```
