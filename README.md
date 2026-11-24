# build-geode-mod-docker

GitHub action to build mods using [geode-docker](https://github.com/geode-sdk/geode-docker) images.

> [!NOTE]
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
        # see note above
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
