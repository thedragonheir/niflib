# Niflib

**C++ library for reading, writing, and manipulating NetImmerse/Gamebryo NIF files.**

Niflib provides a native C++ interface for working with the NIF (NetImmerse File) format used by games and tools built on NetImmerse and Gamebryo technology. It exposes file I/O, object references, scene graph structures, geometry data, and related types for applications that need to inspect or process NIF assets.

This repository is a fork of the original [Niftools/Niflib](https://github.com/niftools/niflib) project.

## Overview

Niflib is a **development library, not a standalone NIF viewer or editor**. It is intended for C++ applications, asset-processing pipelines, format research, and reverse-engineering tools.

The library provides functions to read NIF files into object lists or a rooted object tree, work with their contents through C++ classes, and write them back to disk. Format coverage and correct round-trip behavior depend on the NIF version and the data structures involved.

## Features

- **NIF file I/O** — Read and write supported binary NIF files using C++ functions and streams.
- **Object and scene graph access** — Work with NIF objects, references, inheritance, and parent/child relationships.
- **Geometry and asset structures** — Access meshes, transforms, materials, textures, skinning data, and other supported blocks.
- **Version-aware processing** — Inspect header information and work with structures used by different NetImmerse/Gamebryo versions.
- **Native C++ integration** — Build shared or static libraries for use in other C++ applications.
- **Development tooling** — CMake builds, optional Boost-based tests, and optional Doxygen API documentation.

Niflib does not guarantee full compatibility with every game, NIF version, or vendor-specific extension. Verify the behavior of the specific assets you need to process.

## Repository Structure

```text
niflib/
├── NvTriStrip/            # NVIDIA triangle-strip utilities
├── TriStripper/          # Triangle-strip generation utilities
├── docsys/               # Documentation tooling submodule
├── include/              # Public C++ headers and object definitions
├── qhull/                # Qhull submodule
├── src/                  # Core implementation and generated object code
├── test/                 # C++ tests
├── CMakeLists.txt        # CMake build configuration
├── Doxyfile.in           # Doxygen configuration template
├── niflib.sln            # Legacy Visual Studio solution
├── build.bat             # Windows NMake build helper
├── VERSION               # Project version
├── change_log.txt        # Development and release history
└── license.txt           # Project license
```

The `docsys/` and `qhull/` directories are tracked as Git submodules. Their configured upstream URLs use legacy Git/SSH transports, which may require manual adjustment if you choose to initialize them.

## Getting Started

### Requirements

- A C++ compiler and compatible build environment.
- [CMake](https://cmake.org/) **3.19 or newer**, as specified by this repository's `CMakeLists.txt`.
- Git to clone the repository.

Optional components:

- **Boost 1.75.0** for test configuration (the root CMake file requests this exact version; the tests also require Boost's unit-test framework).
- **Doxygen** to generate API documentation during the CMake build.

### Clone the repository

```bash
git clone https://github.com/thedragonheir/niflib.git
cd niflib
```

The default branch in this fork is `develop`.

### Build with CMake

Configure and build the project from the repository root:

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```

The CMake configuration defines two library targets:

| Target | Description |
| --- | --- |
| `niflib` | Shared library |
| `niflib_static` | Static library (built with `NIFLIB_STATIC_LINK`) |

On multi-configuration generators such as Visual Studio, `--config Release` selects the build configuration. The build output location and filename depend on the selected compiler and generator.

To install the libraries and public headers to a chosen directory:

```bash
cmake --install build --config Release --prefix ./install
```

On Windows, the repository also contains `niflib.sln` and `build.bat` for legacy Visual Studio/NMake workflows. Compatibility with current toolchains is not guaranteed; CMake is the recommended starting point.

## Basic Usage

Include [`niflib.h`](include/niflib.h) in your C++ program and link against the built library. The main file-reading entry points include:

| API | Purpose |
| --- | --- |
| `Niflib::ReadNifList()` | Load NIF blocks into a collection of object references. |
| `Niflib::ReadNifTree()` | Load a NIF object tree from a file or stream. |
| `Niflib::WriteNifTree()` | Write a tree of NIF objects to a file or stream. |
| `Niflib::NifInfo` | Carry file-header information, including version metadata. |

For example, to open a NIF file and report the number of loaded objects:

```cpp
#include <iostream>
#include <exception>
#include "niflib.h"

int main() {
    try {
        Niflib::NifInfo info;
        const auto objects = Niflib::ReadNifList("sample.nif", &info);
        std::cout << "Loaded " << objects.size() << " NIF objects.\n";
    } catch (const std::exception& error) {
        std::cerr << "Unable to read NIF: " << error.what() << '\n';
        return 1;
    }

    return 0;
}
```

Replace `sample.nif` with a file you have permission to use. Configure your application's include paths to point to `include/`, and link to either `niflib` or `niflib_static` as appropriate. When linking statically, define `NIFLIB_STATIC_LINK` in the consuming application as well.

For object-specific operations, consult the headers under [`include/`](include/) and the existing examples in [`test/`](test/). Niflib uses reference types to manage NIF objects and reports many failures through C++ exceptions.

## Testing

When CMake finds the Boost version requested by the project and the required unit-test component, it enables the test suite. Run configured tests with:

```bash
ctest --test-dir build --output-on-failure -C Release
```

If Boost cannot be found, the core libraries can still be configured, but the tests are not added. The tests and build scripts originate from an older codebase, so some adjustments may be necessary on newer compilers or platforms.

## API Documentation

If Doxygen is installed and found during CMake configuration, the build defines an `apidocs` target and generates HTML API documentation:

```bash
cmake --build build --target apidocs
```

The generated documentation is located at `build/doc/html/index.html` for the build directory shown above. The public headers also contain API comments and usage guidance.

## Related Projects

Niflib belongs to the broader [Niftools](https://www.niftools.org/) ecosystem:

- **[NifXML](https://github.com/thedragonheir/nifxml)** — XML-based specification of the NIF file format.
- **[NifSkope](https://github.com/niftools/nifskope)** — Graphical viewer and editor for NIF assets.
- **[PyFFI](https://github.com/niftools/pyffi)** — Python library for processing block-structured binary files.
- **[Blender Niftools Addon](https://github.com/niftools/blender_niftools_addon)** — Blender import/export integration for supported NIF-related formats.

Related tools may support different subsets of the format or target different game versions.

## Contributing

Bug fixes, compatibility improvements, documentation updates, and reproducible test cases are welcome.

For an actionable change:

1. Identify the game, NIF version, and affected object or data type.
2. Include a minimal reproduction or suitable test asset where licensing permits.
3. Document any expected binary-layout or serialization differences.
4. Run the relevant tests when your build environment supports them.
5. Keep backward compatibility in mind, particularly for older file versions.

For fixes intended to benefit the wider community, consider contributing them to the [upstream Niftools/Niflib repository](https://github.com/niftools/niflib).

## Documentation and Resources

- [Niftools website](https://www.niftools.org/)
- [Original Niflib repository](https://github.com/niftools/niflib)
- [Project change log](change_log.txt)
- [Public API header](include/niflib.h)
- [NIF format specification (NifXML)](https://github.com/niftools/nifxml)

## License

Niflib is distributed under a **BSD 3-Clause-style license**. See [`license.txt`](license.txt) for the authoritative copyright notice, conditions, and disclaimer.

Separate third-party dependencies and submodules may have their own license terms.

## Acknowledgments

Niflib was developed by the **Niftools project** and its contributors. This fork preserves the project's origin and builds on the community's work on NetImmerse/Gamebryo file formats and tools.
