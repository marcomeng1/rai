# rai

Core C++ code base for Robotics & AI: data structures, kinematics and
configurations, KOMO motion optimization, optimizers, logic and LGP,
perception, GUI, and the `ry` Python bindings. It contains only sources and
is meant to be used as a submodule of a wrapping project such as
[BotOp](https://github.com/marcomeng1/botop).

This `pixi-cleanup` branch keeps the code of upstream commit `fff7c885` and
modernizes the build: dependencies from a pixi environment instead of `apt` +
`~/.local`, no hardcoded paths in CMake, retired code removed.

## Build

rai is built inside the pixi environment of the wrapping project. In BotOp,
`pixi run install` compiles rai together with BotOp. To build rai standalone
(library, `kinEdit`, `meshTool`, optionally tests) from a BotOp checkout:

```bash
pixi shell
cmake -S rai -B rai/build -G Ninja -DBUILD_TESTS=ON
cmake --build rai/build
```

| CMake option | default | |
|---|---|---|
| `USE_PHYSX` | ON | PhysX simulation |
| `USE_H5` | ON | HDF5 I/O |
| `USE_FREETYPE` | ON | text rendering |
| `USE_OPENCV` | OFF | OpenCV-based perception |
| `BUILD_TESTS` | OFF | build `test/*/*/main.cpp` |

Backends present in the code but not enabled by this build (compiled to
stubs): Bullet, ODE, Featherstone, PCL, Ceres, Ipopt, NLopt, Coal, CUDA, Qt.

`_make/` holds the older Makefile build; it is still used to generate the
per-module `unity.cxx` files and by per-test Makefiles.

## Documentation and related repos

* [robotic Python package documentation](https://marctoussaint.github.io/robotic/)
* [test/](test/): `main.cpp` files are the best entry point into the C++ API
* [BotOp](https://github.com/marcomeng1/botop): robot operation built on rai
  (upstream: [MarcToussaint/botop](https://github.com/MarcToussaint/botop))
* [robotic on PyPI](https://pypi.org/project/robotic/)
* upstream rai: [MarcToussaint/rai](https://github.com/MarcToussaint/rai)
