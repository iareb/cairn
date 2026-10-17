# Cairn

[![CI](https://github.com/iareb/cairn/actions/workflows/ci.yml/badge.svg)](https://github.com/iareb/cairn/actions/workflows/ci.yml)

A minimal Git clone written in C, built to understand how Git works under the hood.

Cairn implements Git's content-addressed object model (blobs, trees, commits) and uses the same on-disk format as Git. A repository written by Cairn can be read by Git, and vice versa.

> **Status:** early development. Nothing beyond `cairn` printing its version works yet.

## Goals

- **Git-compatible on-disk format:** objects, index and refs are byte-for-byte identical to Git's. Git is used as the reference implementation in tests.
- **Own repository directory:** Cairn stores its data in `.cairn/`, which has the same layout as `.git/`.
- **Cross-platform:** builds and runs natively on Linux and Windows (MSVC), without WSL or MSYS2.
- **Same commands as Git** for the subset it implements.

### Planned commands (first version)

| Low-level | Everyday |
| --- | --- |
| `init`, `hash-object`, `cat-file`, `update-index`, `write-tree`, `commit-tree`, `update-ref`, `symbolic-ref` | `add`, `commit`, `log` |

### Out of scope

Networking (`clone`, `fetch`, `push`), merging, packfiles, `.gitignore`, line-ending conversion, symbolic links. `diff` is planned for a later version.

## Building

Requirements:

- CMake 3.21 or newer
- A C17 compiler: MSVC (Visual Studio 2022) on Windows, GCC or Clang on Linux

Configure, build and test with one of the presets (`debug`, `release`, `asan`):

```sh
cmake --preset debug
cmake --build --preset debug
ctest --preset debug
```

The executable ends up in `build/<preset>/`, e.g. `build/debug/Debug/cairn.exe` on Windows.

The `asan` preset builds with AddressSanitizer.

## License

[MIT](LICENSE)
