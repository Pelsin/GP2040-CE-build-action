# GP2040-CE Build Action

A composite GitHub Action that builds [GP2040-CE](https://github.com/OpenStickCommunity/GP2040-CE) firmware (`.uf2` + `.elf`) for a single board config.

The action takes care of cmake, the `arm-none-eabi-gcc` toolchain, `pico-sdk`, building the web configurator (optional) and the CMake configure and build.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `board-config` | yes | | Board config folder name, used as `GP2040_BOARDCONFIG` (e.g. `Pico`, `OpenCore0`). |
| `source-path` | no | `''` | Path (relative to the workspace) to an existing GP2040-CE checkout. When empty, `upstream-repo` is checked out at `upstream-tag` into `./gp2040-ce`. |
| `upstream-tag` | when `source-path` is empty | `''` | GP2040-CE tag/ref to check out. |
| `upstream-repo` | no | `OpenStickCommunity/GP2040-CE` | Repository to check out when `source-path` is empty. |
| `config-path` | no | `''` | Directory containing `BoardConfig.h`, the board header and `assets/`. Copied (minus `.git`/`.github`) into `configs/<board-config>/`, replacing anything already there. |
| `skip-web-build` | no | `false` | When `true`, the web configurator is not built. `lib/httpd/fsdata.c` must already exist in the source, and the action fails if it is missing. |
| `build-type` | no | `Release` | CMake build type. |

## Outputs

| Output | Description |
|---|---|
| `firmware-path` | Absolute path to the built `.uf2`. |
| `elf-path` | Absolute path to the built `.elf` (empty if none was produced). |
| `gp2040-ce-path` | Absolute path to the GP2040-CE source that was built. |

## Usage

Replace `<owner>` with the account that hosts this action, and pin a tag or commit SHA rather than `@main` for anything you rely on.

### Board config repo (release tag + external config)

```yaml
jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
        with:
          path: config

      - id: build
        uses: <owner>/gp2040-ce-build-action@main
        with:
          board-config: MyBoard
          upstream-tag: <gp2040-ce release tag>
          config-path: config

      - uses: actions/upload-artifact@v4
        with:
          name: GP2040-CE-MyBoard
          path: ${{ steps.build.outputs.firmware-path }}
```

Check the config out into a subdirectory (`path: config`), so it does not collide with the `gp2040-ce/` and `pico-sdk/` directories the action creates in the workspace.

## Workspace layout

The action writes these directories into `$GITHUB_WORKSPACE`:

- `gp2040-ce/`: only when `source-path` is empty.
- `pico-sdk/`: always.
- `<source>/build/`: the CMake build directory, which contains the `.uf2` and `.elf`.
