# RAI bare code

This repo contains core sources related to Robotics & AI. Users are
not recommended to use this repo alone.  Please have a look at example
projects that use this bare code as a submodule and expose and explain
some subset of functionalities. Esp. the
[robotic python lib](https://github.com/MarcToussaint/robotic/), which
now co-installs C++ headers and a compiled shared lib.

'bare code' means that this repo contains only sources, a minimal
Ubuntu-specific build system, and development tests. It is mostly used
as submodule in other integrated projects, with their own
out-of-source build system.

## Brief history

Parts of the code have there origin at around 2004 (Edinburgh). The
code grew over the years to a large repo with many projects from all
lab members, but a somewhat consistent scope of code shared between
projects. This repo includes a selection of the code shared between
projects and contains a set of representations and methods for
Robotics, ML and AI. As the functionality is diverse I don't even try
to explain.

## Repos wrapping rai:

* [Robotic Python Lib](https://pypi.org/project/robotic/)
* [BotOp](https://github.com/MarcToussaint/botop) (pixi-based fork: [marcomeng1/botop](https://github.com/marcomeng1/botop))

## Documentation

The there is no proper documentation of the full rai code. I recommend starting with 
* The [robotic python lib documentation](https://marctoussaint.github.io/robotic/), which explains core features (but certainly not the underlying code base),
* With Doxygen (see [rai-maintenence help](https://github.com/MarcToussaint/rai-maintenance/tree/master/help)) you can get an API.
* The [Wiki page](../../wiki) contains an older introduction to KOMO. There is also an older KOMO tech report on arxiv: <https://arxiv.org/abs/1407.0414>
* Eventually, the [test main.cpp files](test/) help really understanding the use of the C++ code base.

## This branch (`pixi-cleanup`)

This fork branch keeps the code of upstream commit `fff7c885` (the
version BotOp is pinned to) and changes only the build setup and dead
code:

* dependencies come from a [pixi](https://pixi.sh) / conda-forge
  environment instead of `apt` + `~/.local` (`_make/install.sh` was removed)
* `rai/CMakeLists.txt` finds everything via `$CONDA_PREFIX`,
  `find_package` and `pkg-config` (no hardcoded `~/.local` or `/usr` paths)
* all `retired/` folders in `src/` and `_make/` were removed (not compiled,
  not referenced; still in git history)
* build fixes: `ry` `addSensor` binding, non-OpenCV `OpencvCamera` stub

## Build

rai has no environment of its own; it is built inside the pixi
environment of a wrapping project, e.g.
[BotOp](https://github.com/marcomeng1/botop) (`pixi.toml`). That
environment provides all libraries from conda-forge and source-builds the
three that conda-forge cannot provide in a usable form
(`scripts/build_deps.sh`): fcl 0.5 (rai uses the non-templated fcl API),
non-reentrant qhull, and PhysX 104.1. Nothing is installed into the
system or `~/.local`.

As part of BotOp (normal use), BotOp's CMake compiles rai directly:

```
git clone --recurse-submodules git@github.com:marcomeng1/botop.git
cd botop
pixi run install
```

Standalone (library, `kinEdit`, `meshTool`, optionally tests), from the
BotOp checkout:

```
pixi shell
cmake -S rai -B rai/build -G Ninja -DBUILD_TESTS=ON
cmake --build rai/build
```

CMake options:

| option | default | |
|---|---|---|
| `USE_PHYSX` | ON | PhysX simulation (needs `pixi run deps` in BotOp) |
| `USE_H5` | ON | HDF5 I/O |
| `USE_FREETYPE` | ON | text rendering in the viewer |
| `USE_OPENCV` | OFF | OpenCV-based perception |
| `BUILD_TESTS` | OFF | build `test/*/*/main.cpp` |

With `BUILD_TESTS=ON`, `test/Core/thread` and `test/Kin/CameraView` do
not compile (they predate the `Var` simplification); all other tests do.

Optional backends that are present in the code but not enabled by any
build here (guarded by `RAI_*` flags, compiled to empty stubs): Bullet,
ODE, Featherstone, PCL, Ceres, Ipopt, NLopt, Coal, CUDA, Qt, JSON.

## Legacy Makefile system

`_make/` contains the older Makefile-based build (`generic.mk`,
`defines.mk`, `config.mk`). It is still used to generate the per-module
`unity.cxx` files (`make unityAll`, called automatically by BotOp's CMake
on first configure) and by the per-test `Makefile`s. Its
`installUbuntu`/`printUbuntu` targets install system packages with
`sudo apt-get` and are not needed with pixi.
